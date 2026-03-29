# Wire Document Store Test Engagement

Test repo for validating the Wire Framework document store feature (Confluence & Notion replication).

## Structure

```
.wire/
├── engagement/
│   └── context.md              # Engagement metadata + docstore config
├── releases/
│   ├── 01-discovery/           # Discovery release (minimal)
│   └── 02-delivery/            # Delivery release (main test target)
│       ├── status.md           # Artifact status + docstore page IDs
│       └── requirements/
│           └── requirements_specification.md   # Pre-written test artifact
└── research/sessions/
```

## Test Steps

### 1. Set up document store
```
/wire:utils-docstore-setup releases/02-delivery
```
Choose Confluence, Notion, or Both. The setup will create a "Wire Documents" parent page.

### 2. Sync the requirements doc
```
/wire:utils-docstore-sync releases/02-delivery requirements
```
Or run the full generate command (which syncs automatically):
```
/wire:requirements-generate releases/02-delivery
```

### 3. Add a comment in Confluence/Notion
Go to the synced page and add a reviewer comment.

### 4. Fetch context for review
```
/wire:utils-docstore-fetch releases/02-delivery requirements
```
Verify the comment appears. Or run the full review command:
```
/wire:requirements-review releases/02-delivery
```

### 5. Edit the page in Confluence/Notion directly
Change a sentence, then re-run the fetch — verify the diff is flagged.

### 6. Re-generate and verify overwrite
```
/wire:requirements-generate releases/02-delivery
```
Confirm the page is updated in place (same page ID, not a new page).
