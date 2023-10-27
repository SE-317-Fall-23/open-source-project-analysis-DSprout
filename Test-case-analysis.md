# Assignment Submission

## Project Selected: <GOhttprouter>

## I. Introduction
- Briefly introduce the purpose of this section, which is to provide a detailed description of two test cases in the selected open-source project.
This section is for the analysis of two test cases within the GOhttprouter project on GitHub. the goal ois to  
The test case selected is the TestRouterUseEscapedPath method in the router_test.go file. the test is for the behavio rof the 
## II. Test Case 1: [TestRouterUseEscapedPath]
### A. Description
This code is a test function for a router implementation that tests whether the router can correctly handle URLs with escaped characters and wildcards in the path. It sets up a route, sends a request with an escaped path, and checks if the route was successfully matched.
### B. Gherkin Syntax (if applicable)
Feature: Router Handling Escaped Paths

  Scenario: Router handles escaped paths and wildcards
    Given a new router is created
    And the router's "UseEscapedPath" property is set to true
    And the boolean variable named "routed" is set to false
    
    When a GET request is made to the path "/user/:name" with the value "foo/bar" for ":name" parameter
    And the router handles the request
    
    Then the route should be successfully matched
    And the route should not fail
### C. Test Steps
Create a new router instance with the property "UseEscapedPath" set to true.

Initialize a boolean "routed" as false.

Define a route in the router to use for the test.

Inside the route handler function, set the "routed" variable to true.

Define an expected parameter "want" and compare to the actual parameters received in the request. If they differ, error

Create a new mock HTTP response writer "w."

Create an HTTP request object "req" with a GET method.

Use the router's "ServeHTTP" method to handle the request using the mock response writer "w."

Check if the "routed" variable is still false after handling the request. If false, error "routing failed"

### D. Code Segments Under Test
func (r *Router) ServeHTTP(w http.ResponseWriter, req *http.Request)
```Java
var data = []struct {
		path string
	}{
		{"/hello/:name"},
		{"/hello/:name/123"},
		{"/hello/:name/234"},
	}

	node := &node{}
	for _, item := range data {
		node.addRoute(item.path, fakeHandler("test"))
	}

	_, _, tsr := node.getValue("/hello/abx/", nil)
	if tsr != true {
		t.Fatalf("want true, is false")
	}
```
### E. Initial State
Create a new router instance with the property "UseEscapedPath" set to true.

Initialize a boolean "routed" as false.

Define a route in the router to use for the test.
### F. Transition
Inside the route handler function, set the "routed" variable to true.

Define an expected parameter "want" and compare to the actual parameters received in the request. If they differ, error

Create a new mock HTTP response writer "w."

Create an HTTP request object "req" with a GET method.

Use the router's "ServeHTTP" method to handle the request using the mock response writer "w."
### G. Expected State
Define an expected parameter "want" and compare to the actual parameters received in the request. If they differ, error : expected is that the params and 'want' are the same

Check if the "routed" variable is still false after handling the request. If false, error "routing failed" : expected value is true

## III. Test Case 2: [TestRedirectTrailingSlash]
### A. Description
The goal of this test is to verify that when a node is configured with specific routes and an attempt is made to retrieve a value for the path "/hello/abx/," the "tsr" (trailing slash redirect) is correctly set to true, indicating that a trailing slash should be redirected.
### B. Gherkin Syntax (if applicable)
Feature: Redirect Trailing Slash

  Scenario: Test Redirecting Trailing Slash
    Given a node with the following routes:
      | Path           |
      | /hello/:name   |
      | /hello/:name/123 |
      | /hello/:name/234 |
    
    When I attempt to retrieve a value for the path "/hello/abx/"
    
    Then the result should indicate that a trailing slash is to be redirected (tsr is true)
### C. Test Steps
Create a data structure that contains a slice of struct elements, each with a "path" attribute of type string.

Create an empty "node" data structure. 

Iterate through each item (route path) in the data slice and add these routes to the "node" data structure.

Attempt to retrieve a value from the "node" data structure for the path "/hello/abx/".

Check whether the retrieved value indicates that a trailing slash should be redirected (tsr).

If the "tsr" is not equal to "true," then raise a testing failure by using "t.Fatalf" to indicate that the expected result (true) does not match the actual result (false).

### D. Code Segments Under Test
```Java
func (n *node) getValue(path string, params func() *Params) (handle Handle, ps *Params, tsr bool) {
walk: // Outer loop for walking the tree
	for {
		prefix := n.path
		if len(path) > len(prefix) {
			if path[:len(prefix)] == prefix {
				path = path[len(prefix):]
				// If this node does not have a wildcard (param or catchAll)
				// child, we can just look up the next child node and continue
				// to walk down the tree
				if !n.wildChild {
					idxc := path[0]
					for i, c := range []byte(n.indices) {
						if c == idxc {
							n = n.children[i]
							continue walk
						}
					}
					// Nothing found.
					// We can recommend to redirect to the same URL without a
					// trailing slash if a leaf exists for that path.
					tsr = (path == "/" && n.handle != nil)
					return
				}
				// Handle wildcard child
				n = n.children[0]
				switch n.nType {
				case param:
					// Find param end (either '/' or path end)
					end := 0
					for end < len(path) && path[end] != '/' {
						end++
					}
					// Save param value
					if params != nil {
						if ps == nil {
							ps = params()
						}
						// Expand slice within preallocated capacity
						i := len(*ps)
						*ps = (*ps)[:i+1]
						(*ps)[i] = Param{
							Key:   n.path[1:],
							Value: path[:end],
						}
					}
					// We need to go deeper!
					if end < len(path) {
						if len(n.children) > 0 {
							path = path[end:]
							n = n.children[0]
							continue walk
						}
						// ... but we can't
						tsr = (len(path) == end+1)
						return
					}
					if handle = n.handle; handle != nil {
						return
					} else if len(n.children) == 1 {
						// No handle found. Check if a handle for this path + a
						// trailing slash exists for TSR recommendation
						n = n.children[0]
						tsr = (n.path == "/" && n.handle != nil) || (n.path == "" && n.indices == "/")
					}
					return
				case catchAll:
					// Save param value
					if params != nil {
						if ps == nil {
							ps = params()
						}
						// Expand slice within preallocated capacity
						i := len(*ps)
						*ps = (*ps)[:i+1]
						(*ps)[i] = Param{
							Key:   n.path[2:],
							Value: path,
						}
					}
					handle = n.handle
					return
				default:
					panic("invalid node type")
				}
			}
		} else if path == prefix {
			// We should have reached the node containing the handle.
			// Check if this node has a handle registered.
			if handle = n.handle; handle != nil {
				return
			}
			// If there is no handle for this route, but this route has a
			// wildcard child, there must be a handle for this path with an
			// additional trailing slash
			if path == "/" && n.wildChild && n.nType != root {
				tsr = true
				return
			}

			if path == "/" && n.nType == static {
				tsr = true
				return
			}

			// No handle found. Check if a handle for this path + a
			// trailing slash exists for trailing slash recommendation
			for i, c := range []byte(n.indices) {
				if c == '/' {
					n = n.children[i]
					tsr = (len(n.path) == 1 && n.handle != nil) ||
						(n.nType == catchAll && n.children[0].handle != nil)
					return
				}
			}
			return
		}
		// Nothing found. We can recommend to redirect to the same URL with an
		// extra trailing slash if a leaf exists for that path
		tsr = (path == "/") ||
			(len(prefix) == len(path)+1 && prefix[len(path)] == '/' &&
				path == prefix[:len(prefix)-1] && n.handle != nil)
		return
	}
}
```
### E. Initial State
Create a data structure that contains a slice of struct elements, each with a "path" attribute of type string.

Create an empty "node" data structure.
### F. Transition
Iterate through each item (route path) in the data slice and add these routes to the "node" data structure.

Attempt to retrieve a value from the "node" data structure for the path "/hello/abx/".

Check whether the retrieved value indicates that a trailing slash should be redirected (tsr).
### G. Expected State
If the "tsr" is not equal to "true," then raise a testing failure by using "t.Fatalf" to indicate that the expected result (true) does not match the actual result (false).

