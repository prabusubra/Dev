# NextJS

## Page vs App Router?
| Feature | Page Router | App Router |
|---|---|---|
| Folder | `pages/` | `app/` |
| Default rendering | Client/server mix | Server Components by default |
| Layouts | Manual/shared layout patterns | Built-in nested layouts |
| Data fetching | `getStaticProps`, `getServerSideProps` | Fetch directly in components |
| Error/loading UI | Manual | Built-in `error.js`, `loading.js` |
| Recommendation | Legacy/common | Modern recommended approach |
