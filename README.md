<div align="center">
	<img src="images/logo.svg" alt="lib-supabase Logo" width="360"/>

# lib-supabase

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

Client package for [Supabase](https://supabase.com/) in [Coi](https://github.com/io-eric/coi).

</div>

This package gives you:

- `Supabase::Credentials` for shared project configuration
- `Supabase::Auth` for sign-in, sign-up, OAuth, and password reset
- `Supabase::Database` for fluent PostgREST queries
- `Supabase::StorageClient` for storage URLs, listing, downloads, remove, and signed URLs

## Install

Copy this package to `.coi/pkgs/supabase` in your project:

```bash
mkdir -p .coi/pkgs
cp -r /path/to/lib-supabase .coi/pkgs/supabase
```

Then import it:

```coi
import "@supabase";
```

## Quick Start

```coi
import "@supabase";

module App;

mut Supabase::Credentials creds = Supabase::Credentials(
	"https://your-project.supabase.co",
	"your-publishable-key"
);

mut Supabase::Auth auth = Supabase::Auth(creds);
mut Supabase::Database db = Supabase::Database(creds);
mut Supabase::StorageClient storage = Supabase::StorageClient(creds);
```

## Core Types

### `Supabase::Credentials`

```coi
pub pod Credentials {
	string url;
	string publishableKey;
}
```

Use one credentials object and pass it to all clients.

## Auth API

Create client:

```coi
mut Supabase::Auth auth = Supabase::Auth(creds);
```

Main methods:

- `signUp(email, password)`
- `signIn(email, password)`
- `signInWithPassword(email, password)`
- `signInWithOAuth(provider, redirectTo)`
- `resetPassword(email)`
- `signOut()`
- `isAuthenticated() : bool`
- `getAccessToken() : string`
- `getPublishableKey() : string`
- `configure(url, publishableKey)`
- `configureCredentials(credentials)`
- `setPublishableKey(key)`

State fields you can read:

- `loading`
- `authenticated`
- `accessToken`
- `userId`
- `userEmail`
- `lastError`
- `lastResponse`

Example:

```coi
auth.signIn("email@example.com", "password");

if (!auth.lastError.isEmpty()) {
	System.error(auth.lastError);
}
```

## Database API

Create client:

```coi
mut Supabase::Database db = Supabase::Database(creds);
```

### Typical query flow

Simple one-liner style:

```coi
db.selectFrom("users", "*", 1);

if (!db.lastError.isEmpty()) {
	System.error(db.lastError);
} else {
	System.log(db.lastResponse);
}
```

Builder style (when you need multiple filters/modifiers):

```coi
db.from("users");
db.limit(1);
db.select("*");

if (!db.lastError.isEmpty()) {
	System.error(db.lastError);
} else {
	System.log(db.lastResponse);
}
```

### Query methods

- `from(table)`
- `select(columns)`
- `selectFrom(table, columns, count)`
- `insert(jsonData)`
- `insertInto(table, jsonData)`
- `update(jsonData)`
- `updateWhereEq(table, column, value, jsonData)`
- `single()`
- `resetQuery()`
- `configure(url, publishableKey)`
- `configureCredentials(credentials)`

### Filters and modifiers

- `eq(column, value)`
- `neq(column, value)`
- `gt(column, value)`
- `lt(column, value)`
- `gte(column, value)`
- `lte(column, value)`
- `like(column, pattern)`
- `ilike(column, pattern)`
- `inList(column, values)`
- `order(column, ascending)`
- `limit(count)`
- `offset(start)`

State fields:

- `loading`
- `lastError`
- `lastResponse`

## Storage API

Create client:

```coi
mut Supabase::StorageClient storage = Supabase::StorageClient(creds);
```

Main methods:

- `from(bucket)`
- `getPublicUrl(path) : string`
- `publicUrl(bucket, path) : string`
- `download(path)`
- `list(path)`
- `listFrom(bucket, path)`
- `remove(path)`
- `removeFrom(bucket, path)`
- `createSignedUrl(path, expiresIn)`
- `createSignedUrlFrom(bucket, path, expiresIn)`
- `configure(url, publishableKey)`
- `configureCredentials(credentials)`

State fields:

- `loading`
- `lastError`
- `lastResponse`

Example:

```coi
storage.from("avatars");
string publicUrl = storage.getPublicUrl("user-123.png");
System.log(publicUrl);
```
## Development

With your Coi app set up in the same workspace, build as usual:

```bash
coi build   or   coi dev
```

## License

MIT License (see [LICENSE](LICENSE)).

