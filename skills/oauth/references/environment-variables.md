# Zoom OAuth Environment Variables

## Standard `.env` keys

| Variable | Required | Used for | Where to find |
| --- | --- | --- | --- |
| `ZOOM_CLIENT_ID` | Yes | General App or S2S client identity | Zoom Marketplace -> General App or S2S app -> App Credentials |
| `ZOOM_CLIENT_SECRET` | Yes | General App or S2S client secret | Zoom Marketplace -> General App or S2S app -> App Credentials |
| `ZOOM_REDIRECT_URI` | General App user/admin OAuth | Authorization code callback | Zoom Marketplace -> General App -> OAuth redirect/allow list |
| `ZOOM_ACCOUNT_ID` | S2S OAuth | Account-level token grant | Zoom Marketplace -> Server-to-Server OAuth app credentials |

## Runtime-only values

- `ZOOM_AUTH_CODE`
- `ZOOM_ACCESS_TOKEN`
- `ZOOM_REFRESH_TOKEN`

Generate these at runtime and keep in secure storage.

## Notes

- Use S2S OAuth where user consent is not required.
- Use Authorization Code flow from a General App when acting on behalf of a Zoom user or admin-authorized account.
- Use `client_credentials` with the General App Client ID and Client Secret when an endpoint requires app-owned scopes.
