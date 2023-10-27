## Project Selected:
4.  GoHTTPRouter (Go)
    -   GitHub Repository: [GoHTTPRouter on GitHub](https://github.com/julienschmidt/httprouter)
    -   Description: GoHTTPRouter is a high-performance HTTP request router for the Go programming language. Its concise codebase makes it a great choice for studying testing in the Go environment.

## I. Introduction
The Testing used in GoHTTPRouter appears to be unit and integration testing for function behavior. Specifically focusing on general important cases such as nil cases or intended interactions between routers or thier peripherals (such as the API). The main goal of the tests is to ensure the methods behave as expected. There is also an extensive number of error results that would allow the tester to identify what went wrong and where. Such as the router_test returning a string identifying why the error occured.

## II. Types of Testing in the Project
### A. Unit Testing
Unit testing is used for ensuring that all methods are behaving as expected. They use the genLongPaths() method to create a non-randomized list of test cases. The path and router tests both use unit testing to ensure they work as intended. For example, the TestPathCleanMallocs test ensures there is no memory leakage, as memory should not be allocated for the CleanPath function in the first place. The framework used for all testing done in GoHTTPRouter is the default testing framework found in GO's standard library. 

### B. Integration Testing
The integration tests for this project are the route_test group of tests. This is because the router itself is utilizing other classes such as http.ResponseWriter. One such tool employed by this is a custom 'not found' handler in the TestRouterNotFound test. Another thing used heavily was mocks, which were used to prevent behavior from outside the intended testing material from influencing the test itself. One example of this was in TestRouterServeFiles, where a mockFileSystem and mockResponseWriter are used, instead of thier 'real' counterparts.

### C. UI (User Interface) Testing
No UI testing for HTTPRouter

## III. Reasons for Choosing These Testing Types
Due to the project working as the backend for a router, there was no need to preform UI testing. As always unit testing was included to ensure all methods performed as expected, such as the methods use to build a tree, or those used to interact with the standard libraries net/http package. The test only used the standard library testing packed, with the exception of the router test which also needed the net/http/httptest package to test iteractions with the net/http package. The technology stack was very small releative to other ptojects, onlt having 3 main files and 3 correseponding test files. The use of static testing in this scenario was more beneficial as the behavior would be predicable, and some tests could utilize a large set of test data such as the data created by genLongPaths(). Certain testing types were also just irrelevant, such as UI testing, due to a lack of presence in the project itself.

## 2. Test Data Generation
### A. Static Test Data
Probably the best example of static tests in this project is in path_test, where there is a function, genLongPaths(),dedicated to generating a large number of test cases for the other tests to use. Both the TestPathCleanLong and BenchmarkPathCleanLong use this static set of data for thier tests.

### B. Dynamic Test Data
There is no dynamic test data, everything uses a statically created series of tests.

## 3. Test Doubles
Both tree_test and router_test utilize test doubles to isolate behavior to that of only themselves. By using the test doubles the tests can simulate the behavior of the Handle or ResponseWriter (respecitvely) without woorying about how they are acutally implimented. This means that it is easier to write focused tests that verify the behavior of the methods rather than potentially having to work around an implimentaion for the Handle or ResponseWriter types. One example of this is the TestTreeAddAndGet test case, where the fakeHandler is used to replace an implimentaition that would normally use a realHandler.

## 4. Discussion on Testing Practices
<!-- 
To find discussions on testing strategy in a GitHub repository, you can follow these steps:

Visit the GitHub repository for the project you're interested in.
Look for the "Issues" tab on the repository's page.
Use the search bar within the Issues tab to search for terms related to testing, such as "testing strategy," "test cases," or "test automation."
-->
Most of the discussions I can find regarding testing are users who have discovered an error and have either developed or want a solution to be developed. I think the follwing issue found here:  https://github.com/julienschmidt/httprouter/issues/322 is a good example of this. User bahlo noticed an error and found a way to consistently create the error, meaning it was reproducable. One of the best things you can do is devise a way to reproduce the error, because you can then use that reproduction method as a test case to ensure the correction for the error works properly. The related commit can be found here : https://github.com/julienschmidt/httprouter/commit/34250257ea144905c752bfaae80d6885f190daf6, and includes both a correction to the code and the corresponding test case to ensure it works properly for others. I think the process of making an error reprocucable is one of the key strategies used in the industry as its allows people to identify where and how the error has occured by tracing the path the reproduced actions take. Another thing I believe to be industry standard is developing a new test for each new method or behavior. The Pull request here : https://github.com/julienschmidt/httprouter/pull/375/files, both adds a new behavior (UseEscapedPath), and when it does so already has a test designed for its implimentation. Creating the test with the method or behavior to ensure that the resulting behavior or method meets the specifications of the origional intent.

