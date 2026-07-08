# Zoom REST API Environment Variables

## Standard `.env` keys

| Variable | Required | Used for | Where to find |
| --- | --- | --- | --- |
| `ZOOM_CLIENT_ID` | Yes | General App or S2S app identity for API access | Zoom Marketplace -> General App or S2S app -> App Credentials |
| `ZOOM_CLIENT_SECRET` | Yes | General App or S2S app secret | Zoom Marketplace -> General App or S2S app -> App Credentials |
| `ZOOM_ACCOUNT_ID` | S2S OAuth mode | Account token grant | Zoom Marketplace -> Server-to-Server OAuth app credentials |
| `ZOOM_REDIRECT_URI` | General App OAuth mode | Authorization callback URL | Zoom Marketplace -> General App -> OAuth redirect/allow list |
| `ZOOM_WEBHOOK_SECRET` | If receiving events | Signature validation for webhook events | Zoom Marketplace -> Event Subscriptions -> Secret Token |
| `ZOOM_API_KEY` | AI Services only | Build-platform JWT issuer | Zoom Build Platform API keys area |
| `ZOOM_API_SECRET` | AI Services only | Build-platform JWT signing secret | Zoom Build Platform API keys area |

## Runtime-only values

- `ZOOM_ACCESS_TOKEN`
- `ZOOM_REFRESH_TOKEN`

## Notes

- Use `ZOOM_ACCOUNT_ID` for server-to-server service integrations.
- General App user/admin integrations require authorization code flow and `ZOOM_REDIRECT_URI`.
- General App app-owned scopes use `client_credentials` with `ZOOM_CLIENT_ID` and `ZOOM_CLIENT_SECRET` and no `ZOOM_ACCOUNT_ID`.
- AI Services endpoints under `/aiservices/*` use Build-platform JWT auth, not the standard REST OAuth token model.
