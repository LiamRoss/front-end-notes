# Jest Testing Framework

Notes on jest syntax and usage with junit XML test results.

> Note: My boilerplate uses automatic test detection within \_\_test\_\_ folders or anything named spec.ts(x) or test.ts(x)

---

## Syntax

### Template

```javascript
describe('Reducer USER_EDIT', () => {		// optional context for all sub tests

	it('should edit user', () => {				// the actual test, must contain an expect
		//  Contents of the test
	}

}
```

---

### Matchers

```javascript
// toBe: uses === to test exact equality
expect(2 + 2).toBe(4)

// toEqual: recursively checks each field of object or array
expect(data).toEqual({one: 1, two: 2})

// not: the opposite of the following matcher
expect(2 + 2).not.toBe(0)
```

For more matchers, visit [Jest Matchers.](https://facebook.github.io/jest/docs/en/using-matchers.html)

## Use with JUnit

#### Dependencies

`npm install --save-dev jest-junit` or `yarn add --dev jest-junit`

#### Information

Allows you to configure a JUnit xml file that contains the results of the latest Jest test run. This is useful if you are running automated tests using a service such as Visual Studio
