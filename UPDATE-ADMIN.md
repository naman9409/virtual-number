# Admin overview update

Stop the existing app with Ctrl+C. Copy the new client/src/App.tsx, client/src/AdminOverview.tsx, client/src/admin.css and server/src/app.js into the corresponding folders in your existing project. Keep your existing server/.env (including PostgreSQL port 5433 and password).

Run npm.cmd run dev and refresh the browser. Log in with your existing admin account. No database reset or new dependencies are needed.

Admin navigation: Business overview, Clients, Verification requests, Payments, Number orders, Login activity. Customers retain their separate workspace.

Payments received counts successful wallet top-ups across all stored records. Number purchase spending is reported separately and is not added to receipts. Seeded wallet credit is not a receipt. Charts show the past 14 days in the database server timezone. Current modes are demo and Razorpay test, not live money.
