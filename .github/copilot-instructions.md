<!--
Repository: project_nghenhac (Node.js / Express / Sequelize)
Purpose: Guidance for AI coding agents to make productive, safe, and repository-aware changes.
--> 

# Copilot instructions for project_nghenhac

Keep responses concise and actionable. When changing code, prefer small, focused edits and run quick checks where possible (lint/build). Reference the exact files below when suggesting or applying changes.

Key facts
- App entry: `src/server.js` (Express 5, Babel) — uses ES modules for most project files but `src/models/index.js` uses CommonJS (Sequelize bootstrap).
- DB: Sequelize with configs in `src/config/config.json` and migrations in `src/migrations`.
- ORM models: `src/models/*.js` and `src/models/index.js` (registers models and associations).
- Routes: `src/routes/web.js` registers server routes and uses `multer` for file uploads (uploads to `uploads/albums` and `uploads/songs`).
- Views: EJS templates in `views/` (server-side rendered pages). Controller logic lives in `src/controllers/homeController.js` which calls service modules under `src/services/`.

What to check before editing
- Verify whether the code targeted uses ES modules (import/export) or CommonJS (require/module.exports). Mixing is intentional in `src/models/index.js` and should be preserved.
- Database migrations: If adding/removing models or columns, update `src/migrations` and provide sequelize-cli commands. The README lists common commands; prefer those exact flags:

```powershell
npx sequelize-cli db:migrate --migrations-path ./src/migrations --config ./src/config/config.json
npx sequelize-cli db:migrate:undo --migrations-path ./src/migrations --config ./src/config/config.json
npx sequelize-cli db:migrate:undo:all --migrations-path ./src/migrations --config ./src/config/config.json
```

Project-specific patterns & conventions
- Services encapsulate DB access and should export an object with async functions (see `src/services/SONGService.js`, `USERService.js`, `ALBUMService.js`). Prefer returning Promises or resolving data rather than throwing raw DB errors.
- Controllers (`src/controllers/homeController.js`) orchestrate request handling, call services, then `res.render()` EJS templates. Keep controller logic thin; heavy DB logic belongs in services.
- File uploads use `multer` configured in `src/routes/web.js`. Uploaded files are stored by filename under `uploads/*` and controllers expect `req.file` or `req.files` and use `.filename`.
- Sessions: `express-session` is configured in `src/server.js` and controllers set `req.session.user` after login/signup.
- Error handling: many services and controllers swallow errors and log to console; when improving, preserve existing user-facing behavior (typically rendering the same page) unless explicitly changing UX.

Build / run / debug
- Start in dev (uses nodemon + babel-node): `npm start` (runs `nodemon --exec babel-node src/server.js`).
- Environment: `.env` is used via `dotenv`. Confirm `PORT` and DB env vars before running.
- Static assets: served from `public/` and `uploads/` (see `src/server.js`). When changing static paths update both `server.js` and any EJS references.

Examples to cite when making changes
- Add a new service method: follow `src/services/SONGService.js` style (async functions, db.* calls, export default object).
- Add a new route with file upload: mirror `router.post('/post-song', UploadSongs.fields(...), homeController.postSong)` in `src/routes/web.js` and handle files in `homeController.postSong`.
- Add a model/migration: use sequelize-cli with flags shown in README and place models in `src/models` and migrations in `src/migrations`.

Safety & tests
- There are no automated tests in this repo. Run the app locally after changes and exercise affected pages (e.g., create/edit song or album) to verify behavior.

When proposing changes
- Explain why the change is needed and reference files/lines (e.g., "change upload path in `src/routes/web.js` and `src/server.js` to `/assets/songs`").
- Provide minimal, focused patches. Avoid large refactors unless user requests them.
- If you must modify the Sequelize bootstrap (`src/models/index.js`) be careful to preserve the CommonJS pattern used there.

If anything is unclear, ask a single targeted question (for example: "Should I convert models to ES modules or keep CommonJS in `src/models/index.js`?").

End of instructions.
