# Why to use async/await

Async await is awesome. It makes working with asynchronous stuff in JS very easy, because it looks like synchronous code. However, I’d recommend learning how Promises work first, because async/await is just syntactic sugar for a Promise, and under the hood it still works like a Promise.

Consider the following promise.
```js
function hashPassword (password) {
	return new Promise((resolve, reject) => {
		const hash = doSomething(password)

		if (hash) {
			resolve(hash)
		} else {
			reject(new Error('Something went wrong..))
		}
	}
}
```

Using async/await it would look like the following.
```js
async function createHash (password) {
	const hash = await hashPassword(password)

	// do something with the hash
}

createHash('cat')
```

This does however not handle an error. To fix this we need to add a `try catch` block.
```js
async function createHash (password) {
	try {
		const hash = await hashPassword(password)
		
		// do something with the hash
	} catch (err) {
		console.error(err)
	}
}

createHash('cat')
```

Simple as that!

You may think that this is much more code to do the same thing as the promise. And you would be right. But what if you were to make more than one asynchronous call in a function?

I’m using a function from [one of my own projects](https://github.com/jeroentvb/windsurf-stats) as an example.

Consider the following function `storeInDb()`. 
```js
function storeInDb (req, res, user) {
	// hashPassword returns a Promise which resolves into a password hash
  hashPassword(user.password)
    .then(hash => {
		// db.query executes a MySql query and returns a Promise with the result
      db.query('INSERT INTO windsurfStatistics.users SET ?', {
        username: user.name,
        email: user.email,
        password: hash
      })
        .then(result => {
			// Get the the previously inserted data AND the generated userId
          db.query('SELECT id FROM windsurfStatistics.users WHERE email = ?', user.email)
            .then(result => {
              let userId = result[0].id
				
				// Set a cookie
              req.session.user = {
                name: user.name,
                email: user.email,
                id: userId
              }

              res.render('setPrefs')
            })
            .catch(err => console.error(err))
        })
        .catch(err => console.error(err))
    })
    .catch(err => console.error(err))
}
```
A brief overview of what happens here:
1. Hash a password
2. Use that hash and insert account details in the database
3. Make a new request to get the previous inserted data AND the automatically generated userId (I know this gets returned in the previous request, I’ve changed it in my project).
4. Store the name, email and userId in a cookie for further use.

I know this is horrible code, because it’s essentially callback-hell because of the nested promise.then()’s.
I did this because I needed to make multiple database queries, but the first query needed to be completed before the second was executed.

Let’s take a look at the above code, but using async/await.

```js
async function storeInDb (req, res, user) {
  try {
    const hash = await hashPassword(user.password)

    await db.query('INSERT INTO windsurfStatistics.users SET ?', {
      username: user.name,
      email: user.email,
      password: hash
    })

    const userData = await db.query('SELECT id FROM windsurfStatistics.users WHERE email = ?', user.email)
    const userId = userData[0].id

    req.session.user = {
      name: user.name,
      email: user.email,
      id: userId
    }

    res.render('setPrefs')
  } catch (err) {
    console.error(err)

    render.unexpectedError(res)
  }
}
```

This is much shorter and much more readable. Awesome!

If a promise resolves in a value you can capture it in a variable using:
```js
const foo = await doSomething()
```

If it doesn’t return a value, or you don’t need the value it returns but you still need to wait before it’s done, you can just do the following:
```js
await doSometing()
```

## Conclusion
Async / await is awesome. It makes code more readable because it looks like synchronous code.  