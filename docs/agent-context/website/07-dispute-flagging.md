# Task: Dispute Flagging

## Depends on
- `website/06-dashboard-submissions-votes` (submission and cluster views in dashboard)

## Goal
Add the ability for users to flag bad canonicalization or incorrect cluster assignment from their dashboard. Disputes are logged and reviewed by operators.

## Files to create/modify

- `web/components/DisputeButton.tsx` — flag button + reason form
- `web/components/DisputeStatus.tsx` — shows dispute status on flagged items
- Update `web/components/SubmissionCard.tsx` — add dispute button
- `web/messages/fa.json` — add dispute translations
- `web/messages/en.json` — add dispute translations

## Specification

### DisputeButton component

A button that opens a modal or inline form for submitting a dispute.

```typescript
interface DisputeButtonProps {
  entityType: "canonicalization" | "cluster_assignment";
  entityId: string;           // submission_id or cluster_id
  disabled?: boolean;         // True if already disputed
}
```

When clicked:
1. Show a form with:
   - **Dispute type**: "The AI misunderstood my submission" or "My submission is in the wrong group" (radio buttons)
   - **Reason**: Free text field (optional but encouraged)
   - Submit button
2. On submit: POST to `/api/user/flag` with `{ entity_type, entity_id, dispute_type, reason }`
3. On success: Show confirmation and update the UI to show dispute status
4. On error: Show error message

### DisputeStatus component

Shows the current state of a dispute:

```typescript
interface DisputeStatusProps {
  status: "dispute_open" | "dispute_resolved" | null;
  resolvedAt?: string;
  resolution?: string;
}
```

Display:
- `dispute_open`: "🔍 در حال بررسی" / "Under review" (yellow badge)
- `dispute_resolved`: "✓ بررسی شد" / "Reviewed" (green badge) + resolution text if available
- `null`: No dispute — show the DisputeButton

### Integration with SubmissionCard

On each submission in the dashboard:
- If `status = "processed"` and no dispute: show DisputeButton for canonicalization
- If submission is in a cluster and no cluster dispute: show DisputeButton for cluster assignment
- If dispute exists: show DisputeStatus instead of button

### API call

```typescript
export async function flagDispute(data: {
  entity_type: "canonicalization" | "cluster_assignment";
  entity_id: string;
  dispute_type: string;
  reason?: string;
}): Promise<void> { ... }
```

### Dispute rules (from frozen decisions)

- Disputed items are tagged but NEVER removed or suppressed
- Operator reviews within 72 hours
- Resolution is by re-running the pipeline or adjusting parameters, NOT by manual content override
- Resolution is logged to the evidence store

## Constraints

- Disputes require authentication. Only the user who submitted can dispute their own submission's canonicalization.
- Users can dispute cluster assignment for any cluster their submission is in.
- A submission can only have one open dispute at a time.
- Dispute submission is rate-limited (inherit from general rate limits).
- The dispute reason is optional — users should be able to flag without explaining (lower friction).

## Tests

Write tests covering:
- DisputeButton renders and is clickable
- Clicking opens the dispute form
- Form submits to API with correct payload (mock fetch)
- Success state: button replaced by DisputeStatus showing "under review"
- Error state: error message shown, form still available
- Already-disputed item shows DisputeStatus instead of button
- Resolved dispute shows resolution status
- DisputeButton disabled when already disputed
- Form works in Farsi (RTL) and English (LTR)
