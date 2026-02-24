<div align="center">
	<img src="images/logo.svg" alt="coi-supabase Logo" width="360"/>

# Supabase for Coi

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

Client package for [Supabase](https://supabase.com/) in [Coi](https://github.com/coi/coi).

</div>

This package gives you:

- `Supabase::Credentials` for shared project configuration
- `Supabase::AuthClient` for sign-in, sign-up, OAuth, and password reset
- `Supabase::DatabaseClient` for fluent PostgREST queries
- `Supabase::StorageClient` for storage URLs, listing, downloads, remove, and signed URLs
- `Supabase::RealtimeClient` for live Postgres change subscriptions over WebSocket

## Install

Install from the Coi registry:

```bash
coi add @coi/supabase
```

Then import it:

```coi
import "@coi/supabase";
```

## Quick Start

```coi
import "@coi/supabase";

module App;

mut Supabase::Credentials creds = Supabase::Credentials(
	"https://your-project.supabase.co",
	"your-publishable-key"
);

mut Supabase::AuthClient auth = Supabase::AuthClient(creds);
mut Supabase::DatabaseClient db = Supabase::DatabaseClient(creds);
mut Supabase::StorageClient storage = Supabase::StorageClient(creds);
mut Supabase::RealtimeClient realtime = Supabase::RealtimeClient(creds);
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

Create a client:

```coi
mut Supabase::AuthClient auth = Supabase::AuthClient(creds);
```

Main methods:

- `onSuccess(handler)`
- `onError(handler)`
- `onAuthSuccess(handler)`
- `onAuthError(handler)`
- `signUp(email, password)`
- `signIn(email, password)`
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
def handleAuthSuccess(string response) : void {
	System.log(response);
}

def handleAuthError(string error) : void {
	System.error(error);
}

auth.onSuccess(&handleAuthSuccess);
auth.onError(&handleAuthError);

auth.signIn("email@example.com", "password");

if (!auth.lastError.isEmpty()) {
	System.error(auth.lastError);
}
```

## Database API

Create a client:

```coi
mut Supabase::DatabaseClient db = Supabase::DatabaseClient(creds);
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
def handleDbSuccess(string response) : void {
	System.log(response);
}

def handleDbError(string error) : void {
	System.error(error);
}

db.onSuccess(&handleDbSuccess);
db.onError(&handleDbError);

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

- `onSuccess(handler)`
- `onError(handler)`
- `onSelectSuccess(handler)`
- `onSelectError(handler)`
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

Create a client:

```coi
mut Supabase::StorageClient storage = Supabase::StorageClient(creds);
```

Main methods:

- `onSuccess(handler)`
- `onError(handler)`
- `onStorageSuccess(handler)`
- `onStorageError(handler)`
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
def handleStorageSuccess(string response) : void {
	System.log(response);
}

def handleStorageError(string error) : void {
	System.error(error);
}

storage.onSuccess(&handleStorageSuccess);
storage.onError(&handleStorageError);

storage.from("avatars");
string publicUrl = storage.getPublicUrl("user-123.png");
System.log(publicUrl);
```

## Realtime API

Create a client:

```coi
mut Supabase::RealtimeClient realtime = Supabase::RealtimeClient(creds);
```

Main methods:

- `connect()`
- `disconnect()`
- `reconnect()`
- `subscribeToTable(schema, table, event)`
- `unsubscribe()`
- `ping()`
- `onEvent(handler)`
- `onError(handler)`
- `configure(url, publishableKey)`
- `configureCredentials(credentials)`

Note: Realtime uses `onEvent` + `onError` (no `onSuccess` callback).

State fields:

- `connected`
- `loading`
- `eventCount`
- `lastEvent`
- `lastError`

Example:

```coi
def handleRealtimeEvent(string payload) : void {
	System.log(payload);
}

def handleRealtimeError(string error) : void {
	System.error(error);
}

realtime.onEvent(&handleRealtimeEvent);
realtime.onError(&handleRealtimeError);

// Subscribe to all row changes in public.messages
realtime.subscribeToTable("public", "messages", "*");
```

Event filter examples for `subscribeToTable`:

- `"*"` (all events)
- `"INSERT"`
- `"UPDATE"`
- `"DELETE"`

> Note: Enable Realtime for your table in Supabase first, otherwise no row-change events will be emitted.

## Examples by API

### 1) Credentials (shared setup)

```coi
import "@coi/supabase";

module App;

mut Supabase::Credentials creds = Supabase::Credentials(
	"https://your-project.supabase.co",
	"your-publishable-key"
);

mut Supabase::AuthClient auth = Supabase::AuthClient(creds);
mut Supabase::DatabaseClient db = Supabase::DatabaseClient(creds);
mut Supabase::StorageClient storage = Supabase::StorageClient(creds);
mut Supabase::RealtimeClient realtime = Supabase::RealtimeClient(creds);
```

### 2) Auth (sign up, sign in, oauth, reset, sign out)

```coi
import "@coi/supabase";

module App;

component AuthDemo {
	mut Supabase::Credentials creds = Supabase::Credentials(
		"https://your-project.supabase.co",
		"your-publishable-key"
	);
	mut Supabase::AuthClient auth = Supabase::AuthClient(creds);

	def handleAuthSuccess(string response) : void {
		System.log(response);
	}

	def handleAuthError(string error) : void {
		System.error(error);
	}

	def run() : void {
		auth.onSuccess(&handleAuthSuccess);
		auth.onError(&handleAuthError);

		auth.signUp("new-user@example.com", "password123");
		auth.signIn("new-user@example.com", "password123");

		auth.signInWithOAuth("github", "http://localhost:8000/auth/callback");
		auth.resetPassword("new-user@example.com");

		if (!auth.lastError.isEmpty()) {
			System.error(auth.lastError);
		}

		if (auth.isAuthenticated()) {
			System.log("Signed in");
			System.log(auth.getAccessToken());
		}

		auth.signOut();
	}
}
```

### 3) Database (select, filters, insert, update)

```coi
import "@coi/supabase";

module App;

component DatabaseDemo {
	mut Supabase::Credentials creds = Supabase::Credentials(
		"https://your-project.supabase.co",
		"your-publishable-key"
	);
	mut Supabase::DatabaseClient db = Supabase::DatabaseClient(creds);

	def handleDbSuccess(string response) : void {
		System.log(response);
	}

	def handleDbError(string error) : void {
		System.error(error);
	}

	def run() : void {
		db.onSuccess(&handleDbSuccess);
		db.onError(&handleDbError);

		// Simple one-liner query
		db.selectFrom("users", "id,name,email", 10);

		// Builder query with filters/modifiers
		db.from("users");
		db.ilike("email", "%@example.com");
		db.order("id", true);
		db.limit(5);
		db.select("id,name,email");

		// Insert row
		db.insertInto("users", `{"name":"Ava","email":"ava@example.com"}`);

		// Update row (where id == 1)
		db.updateWhereEq("users", "id", "1", `{"name":"Ava Updated"}`);

		if (!db.lastError.isEmpty()) {
			System.error(db.lastError);
		} else {
			System.log(db.lastResponse);
		}
	}
}
```

### 4) Storage (public urls, list, download, remove, signed url)

```coi
import "@coi/supabase";

module App;

component StorageDemo {
	mut Supabase::Credentials creds = Supabase::Credentials(
		"https://your-project.supabase.co",
		"your-publishable-key"
	);
	mut Supabase::StorageClient storage = Supabase::StorageClient(creds);

	def handleStorageSuccess(string response) : void {
		System.log(response);
	}

	def handleStorageError(string error) : void {
		System.error(error);
	}

	def run() : void {
		storage.onSuccess(&handleStorageSuccess);
		storage.onError(&handleStorageError);

		storage.from("avatars");

		string url = storage.getPublicUrl("users/user-1.png");
		System.log(url);

		storage.list("users");
		storage.download("users/user-1.png");
		storage.createSignedUrl("users/user-1.png", 3600);
		storage.remove("users/user-1.png");

		// One-call helper
		string sameUrl = storage.publicUrl("avatars", "users/user-2.png");
		System.log(sameUrl);

		if (!storage.lastError.isEmpty()) {
			System.error(storage.lastError);
		} else {
			System.log(storage.lastResponse);
		}
	}
}
```

### 5) Realtime (live row updates)

```coi
import "@coi/supabase";

module App;

component RealtimeDemo {
	mut Supabase::Credentials creds = Supabase::Credentials(
		"https://your-project.supabase.co",
		"your-publishable-key"
	);
	mut Supabase::RealtimeClient realtime = Supabase::RealtimeClient(creds);

	def handleRealtimeEvent(string payload) : void {
		System.log(payload);
	}

	def handleRealtimeError(string error) : void {
		System.error(error);
	}

	mount {
		realtime.onEvent(&handleRealtimeEvent);
		realtime.onError(&handleRealtimeError);

		// Subscribe to all changes on public.messages
		realtime.subscribeToTable("public", "messages", "*");
	}

	def cleanup() : void {
		realtime.unsubscribe();
		realtime.disconnect();
	}
}
```

## Development

With your Coi app set up in the same workspace, build as usual:

```bash
coi build   or   coi dev
```

## License

MIT License (see [LICENSE](LICENSE)).

