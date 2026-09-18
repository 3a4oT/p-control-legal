# p-control-legal

Two redirects, and nothing else. The privacy policy, the terms of use and the account-deletion
page of [p-control](https://github.com/3a4oT/p-control-android) live in `p-control-server`'s
`web/site/legal` since 2026-09-18 and are served at:

- https://control.rovenskyi.com/uk/privacy · https://control.rovenskyi.com/en/privacy
- https://control.rovenskyi.com/uk/terms · https://control.rovenskyi.com/en/terms
- https://control.rovenskyi.com/uk/delete-account · https://control.rovenskyi.com/en/delete-account

This repository is kept so that every address ever printed on a Play listing or encoded in a
television's QR still resolves:

- `index.html` — redirects to `/en/privacy`.
- `delete-account.html` — redirects to `/en/delete-account`.

The draft Play Console answers (`data-safety-form.md`, `permissions-declaration.md`) moved to
`p-control-server/docs/play/`. Nothing is edited here any more; the `updating-the-privacy-policy`
skill and `p-control-server/docs/superpowers/specs/2026-09-18-legal-pages-on-our-domain-design.md`
say where and how the text changes.
