# Foreground-app reporting (optional)

The screenshot feature works on its own. If you *also* want your AI to see which
app is currently open (without a screenshot), feed the server a stream of
foreground-app events. On Android the simplest way is [MacroDroid](https://www.macrodroid.com/).

This is optional and independent of the screenshot flow.

## The macro

1. **Trigger:** *Application Launched* → *Any application*.
2. **Action:** *HTTP Request* →
   - Method: `POST`
   - URL: `https://your-domain/phone/activity?token=YOUR_TOKEN`
   - Content type: `application/json`
   - Body:
     ```json
     {"app": "{app_name}", "event": "switch"}
     ```
     (`{app_name}` is a MacroDroid magic-text variable for the launched app.)

That's it. The server keeps the last 2 hours of events (max 15), and `peek.py`
prints them:

```
$ python3 peek.py
most recent: WeChat (2m ago)
----------------------------
      2m ago  WeChat
     14m ago  Chrome
     40m ago  Spotify
```

## Notes

- Only the **app name** is sent — never contents. If you want to see contents, that's what a screenshot is for.
- The token goes in the URL query string here because MacroDroid's HTTP action makes custom headers fiddly; the server accepts the token either way.
- No MacroDroid? Any automation tool that can fire an HTTP POST on app-switch works the same way.
