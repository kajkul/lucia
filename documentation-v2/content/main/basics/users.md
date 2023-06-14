---
order: 0
title: "Users"
description: "Learn about users in Lucia"
---

Users are stored in your database and are represented by a `User` object within Lucia. By default, they only hold their user id.

```ts
const user = {
	userId: "laRZ8RgA34YYcgj"
};
```

#### User id

The primary way to identify users is by their user id. It's randomly generated by Lucia and it's 15 characters long. You can of course configure it to [generate your own user id]().

#### User attributes

You can store additional data to your users by defining user attributes, such as email and username.

## Defining user attributes

User attributes are inferred by the return type of [`Configuration.getUserAttributes()`](). The function takes in the user object directly from the user table.

```ts
import { lucia } from "lucia";

lucia({
	// ...

	// setting params type as an example - DO NOT DO THIS
	// argument `databaseUser` is automatically typed
	// see next section
	getUserAttributes: (databaseUser: DatabaseUser) => {
		return {
			username: databaseUser.username
		};
	}
});

type DatabaseUser = {
	id: string;
	username: string; // 'username' column in user table
};

// expected `User` object
type User = {
	userId: string; // always included
	username: string; // return type of `getUserAttributes()`
};
```

You can add additional columns to the user table so long as the `id` column exists as primary key. In the example above, the `username` column is added to the table and is included in `databaseUser`.

### Typing database user attributes

You can define database user attributes with `Lucia.DatabaseUserAttributes`. You cannot make any of the field optional.

```ts
/// <reference types="lucia" />
declare namespace Lucia {
	// ...
	type DatabaseUserAttributes = {
		// id should not be defined here
		username: string;
	}; // => `username` included in `getUserAttributes()` argument
}
```

## Create users

You can create new users by calling [`Auth.createUser()`]().

```ts
import { auth } from "./lucia.js";

try {
	await auth.createUser({
		key: {
			providerId,
			providerUserId,
			password
		},
		attributes: {
			username
		} // expects `Lucia.DatabaseUserAttributes`
	});
} catch {
	// key already exists
	// or unique constraint failed for user attributes
}
```

The fields of the `attributes` property is whatever you defined in `Lucia.DatabaseUserAttributes`. It should be an empty object with the default configuration (no user attributes defined).

You can optionally create a [key]() alongside the user. This is preferably compared to creating a key on its own as both the user and key will not be created if one fails (due to duplicate data etc). It will throw `AUTH_DUPLICATE_KEY_ID` if the key already exists. To not create a key, pass `null`:

```ts
await auth.createUser({
	key: null
	// ...
});
```

## Update user attributes

You can update attributes of a user with [`Auth.updateUserAttributes()`](). You can update a single field or multiple fields. It returns the user of the updated user, or throws `AUTH_INVALID_USER_ID` if the user does not exist.

> (red) **Make sure to invalidate all sessions of the user on password or privilege level change.** You can create a new session to prevent the current user from being logged out.

```ts
import { auth } from "./lucia.js";

try {
	const user = await auth.updateUserAttributes(
		userId,
		{
			username: newUsername
		} // expects partial `Lucia.DatabaseUserAttributes`
	);
} catch {
	// invalid user id
	// or unique constraint failed for user attributes
}
```

```ts
const user = await auth.updateUserAttributes(userId, {
	role: "admin" // new privileges
});
await auth.invalidateAllUserSessions(user.userId); // invalidate all user sessions => logout all sessions
const session = await auth.createSession(user.userId); // new session
// store new session
```

## Delete users

You can delete users with [`Auth.deleteUser()`](). All sessions and keys of the user will be deleted as well. This method will succeed regardless of the validity of the user id.

```ts
import { auth } from "./lucia.js";

await auth.deleteUser(userId);
```