# Match import and recap

An older system POSTs the payload below to an API endpoint. It is one completed match, sent once after the match finished. Take it in, store it, and show a small recap page that support and organizers can use.

The stack is Laravel and Vue. Everything else is your choice.

## Before you code

Explain how you would model this. We mostly care about:

- how you receive and read the payload at the endpoint (validation, idempotency)
- the data model you pick
- which fields you store as separate columns
- what you keep as raw payload
- how you calculate the recap (derived or stored)
- how you test the import

A few things to notice in the payload:

- Each team inside a leg has its own turns array. They are not interleaved.
- A turn's `player_index` is the index into that team's `players` array.
- The recap can be small or detailed. Score, winner and duration is enough. Per-team or per-player stats are nice if there is time.

Auth on the endpoint and live updates are out of scope. Assume the request is trusted. Happy to talk about both if you want to.

## The payload

```json
{
    "external_id": "match_abc123",
    "tournament_id": "dutch-open-2026",
    "event": {
        "id": "pairs-open",
        "name": "Pairs Open"
    },
    "board": 12,
    "round": "Quarter Final",
    "status": "completed",
    "started_at": "2026-05-24T13:05:00Z",
    "finished_at": "2026-05-24T13:32:00Z",
    "options": {
        "type": "01",
        "starting_score": 501,
        "double_out": true,
        "double_in": false
    },
    "teams": [
        {
            "id": "team_1",
            "name": "Smith / Wright",
            "players": [
                {
                    "id": "p1",
                    "name": "Michael Smith",
                    "country": "ENG"
                },
                {
                    "id": "p2",
                    "name": "Peter Wright",
                    "country": "SCO"
                }
            ]
        },
        {
            "id": "team_2",
            "name": "Van Barneveld / Van Gerwen",
            "players": [
                {
                    "id": "p3",
                    "name": "Raymond van Barneveld",
                    "country": "NED"
                },
                {
                    "id": "p4",
                    "name": "Michael van Gerwen",
                    "country": "NED"
                }
            ]
        }
    ],
    "legs": [
        {
            "number": 1,
            "competitors": [
                {
                    "team_id": "team_2",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 60,  "current_score": 441, "color": null },
                        { "player_index": 1, "darts_thrown": 3, "points": 85,  "current_score": 356, "color": null },
                        { "player_index": 0, "darts_thrown": 3, "points": 100, "current_score": 256, "color": "TON" },
                        { "player_index": 1, "darts_thrown": 3, "points": 55,  "current_score": 201, "color": null },
                        { "player_index": 0, "darts_thrown": 3, "points": 56,  "current_score": 145, "color": null }
                    ]
                },
                {
                    "team_id": "team_1",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 100, "current_score": 401, "color": "TON" },
                        { "player_index": 1, "darts_thrown": 3, "points": 140, "current_score": 261, "color": "TON 40" },
                        { "player_index": 0, "darts_thrown": 3, "points": 100, "current_score": 161, "color": "TON" },
                        { "player_index": 1, "darts_thrown": 3, "points": 121, "current_score": 40,  "color": "TON" },
                        { "player_index": 0, "darts_thrown": 1, "points": 40,  "current_score": 0,   "color": null }
                    ]
                }
            ]
        },
        {
            "number": 2,
            "competitors": [
                {
                    "team_id": "team_1",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 81,  "current_score": 420, "color": null },
                        { "player_index": 1, "darts_thrown": 3, "points": 60,  "current_score": 360, "color": null },
                        { "player_index": 0, "darts_thrown": 3, "points": 100, "current_score": 260, "color": "TON" },
                        { "player_index": 1, "darts_thrown": 3, "points": 96,  "current_score": 164, "color": null },
                        { "player_index": 0, "darts_thrown": 3, "points": 100, "current_score": 64,  "color": "TON" }
                    ]
                },
                {
                    "team_id": "team_2",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 95,  "current_score": 406, "color": "95!" },
                        { "player_index": 1, "darts_thrown": 3, "points": 100, "current_score": 306, "color": "TON" },
                        { "player_index": 0, "darts_thrown": 3, "points": 85,  "current_score": 221, "color": null },
                        { "player_index": 1, "darts_thrown": 3, "points": 140, "current_score": 81,  "color": "TON 40" },
                        { "player_index": 0, "darts_thrown": 2, "points": 81,  "current_score": 0,   "color": null }
                    ]
                }
            ]
        },
        {
            "number": 3,
            "competitors": [
                {
                    "team_id": "team_2",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 45, "current_score": 456, "color": null },
                        { "player_index": 1, "darts_thrown": 3, "points": 60, "current_score": 396, "color": null },
                        { "player_index": 0, "darts_thrown": 3, "points": 85, "current_score": 311, "color": null },
                        { "player_index": 1, "darts_thrown": 3, "points": 79, "current_score": 232, "color": null }
                    ]
                },
                {
                    "team_id": "team_1",
                    "turns": [
                        { "player_index": 0, "darts_thrown": 3, "points": 140, "current_score": 361, "color": "TON 40" },
                        { "player_index": 1, "darts_thrown": 3, "points": 100, "current_score": 261, "color": "TON" },
                        { "player_index": 0, "darts_thrown": 3, "points": 121, "current_score": 140, "color": "TON" },
                        { "player_index": 1, "darts_thrown": 3, "points": 140, "current_score": 0,   "color": null }
                    ]
                }
            ]
        }
    ]
}
```

## Practical notes

- Work as you normally would. Think out loud where it helps.
- Use the Laravel and Vue versions you are comfortable with.
- Database is your choice. Be ready to explain why.
- Small and finished beats large and half done. "I would do X next" is a fine answer.
- Ask questions any time. Treat us as the product owner.
