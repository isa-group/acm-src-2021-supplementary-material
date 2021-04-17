# *ICSE: G: Automated Management of Inter-Parameter Dependencies in Web APIs* - Supplementary material


## Data location

All resources are placed inside the `target` directory. The `<EVALUATION_NAME>` identifies the experiment in terms of the service tested and how the test cases were generated. For example, `youtubeSearch_allNominalIgnoreDependencies` means that the YouTube-Search service was tested with 100% nominal test cases generated with RT, i.e., ignoring inter-parameter dependencies.

For each experiment performed, the following resources are provided:
  - **Allure reports**. Location: `target/allure-reports/<EVALUATION_NAME>/index.html`. Graphical interface to visualize all test cases run on the API, which of them failed, and why. **NOTE**: Allure HTML files cannot be rendered locally, you must use a server for such purpose. Advice: open the files from an IDE such as IntelliJ IDEA.
  - **Test cases**. Location: `target/test-data/<EVALUATION_NAME>/test-cases.csv`. These are a representation of the HTTP requests sent to the API, without the response.
  - **Test results**. Location: `target/test-data/<EVALUATION_NAME>/test-results.csv`. These are a representation of the HTTP responses sent back by the API.
  - **Time report**. Location: `target/test-data/<EVALUATION_NAME>/time.csv`. Includes information on the time taken for the whole experiment, the generation and the execution of test cases.
  - **Test coverage**. Location: `target/coverage-data/<EVALUATION_NAME>/test-coverage.json`. Test coverage achieved by the test cases, according to the criteria defined in [Test Coverage Criteria for RESTful Web APIs](https://dl.acm.org/doi/10.1145/3340433.3342822).


## Bugs found

Here we detail the bugs found in each service.

### Foursquare - Search venues
  - *500 status code, non-reproducible*. A faulty request obtained a server error. **Test ID**: `test_rh8b1qswv9di_searchVenues`.
  - *Error response after valid request, reproducible*. Multiple valid requests obtained the error `Must provide parameters (ll and radius) or (sw and ne) or (near and radius)` even when those conditions met. **Test ID**: `test_1h7o4yd4erol5_searchVenues`.
  - *Error response after valid request, reproducible*. Multiple valid requests obtained the error `Must provide parameter query` even though all dependencies were satisfied and the parameter `query` was not involved in any of them. **Test ID**: `test_1hrrikxnk2tlt_searchVenues`.
  - *Successful response after request violating dependency, reproducible*. Numerous requests violating dependencies obtain successful responses. In fact, we had to modify all dependencies present in this service, as they were incorrecly described in the documentation. **Test ID**: `test_1h7fwrcprxbhd_searchVenues`.
  - *Successful response after request with faulty parameter, reproducible*. Requests using a `radius` higher than the maximum allowed (100000) obtain successful responses. **Test ID**: `test_1h85apg914zea_searchVenues`.
  - *Successful response after request with faulty parameter, reproducible*. Requests using a `limit` higher than the maximum allowed  (50) obtain successful responses. **Test ID**: `test_1hr5dv8umqskx_searchVenues`.
  - *Successful response after request with faulty parameter, reproducible*. Requests using disallowed values for the parameter `intent` (enum) obtain successful responses. **Test ID**: `test_1hrj8qnymvub7_searchVenues`.
  - *OAS validation failure, reproducible*. When removing the required parameter `v`, the status code 410 is returned, but such status code is not documented in the API specification. **Test ID**: `test_1h7gdic67ko4w_searchVenues`.
  - *OAS validation failure, reproducible*. Some results returned by the API include properties not defined in the API specification, namely: `response.confident`, `response.geocode`, `response.venues[].hasPerk"`, `response.venues[].referralId`, `response.venues[].location.neighborhood`. **Test ID**: `test_1ib8pxoxamhx3_searchVenues`.

### GitHub - Get user repos
  - *Successful response after request with faulty parameter, reproducible*. Any request that satisfies inter-parameter dependencies but uses invalid values for any parameter (e.g., an integer value for the parameter `sort`) obtains a successful response. This seems to be due to the fact that the service ignores any parameter using an invalid value. **Test ID**: `test_1ib99sznvaxbr_getUserRepos`.

### Stripe - Create coupon
  - *Error response after valid request, reproducible*. The Stripe documentation states that at least one of the parameters `amount_off` or `percent_off` must be used. However, those parameters are actually mutually exclusive. This causes requests that are supposedly valid to be rejected. **Test ID**: `test_1h7geaqmn6lo5_PostCoupons`.

### Stripe - Create product
  - *Error response after valid request, reproducible*. There exist two unspecified dependencies in this service. They cause one of every two supposedly valid requests to be rejected.  **Test ID**: `test_1h7fvyyo3bo12_PostProducts`.

### Tumblr - Get blog likes
  - *Successful response after request violating dependency, reproducible*. Numerous requests violating dependencies obtain successful responses. This is because the dependencies of the service are actually 'softer' than what is described in the documentation, i.e., more parameters combinations are allowed. **Test ID**: `test_1h7geaq9xlvtu_getBlogLikes`.
  - *Successful response after request with faulty parameter, reproducible*. Similarly to GitHub, any request that satisfies inter-parameter dependencies but uses invalid values for any parameter (e.g., a string value for the parameter `before`) obtains a successful response. This seems to be due to the fact that the service ignores any parameter using an invalid value. **Test ID**: `test_1h7qzgkp2asz8_getBlogLikes`.
  - *OAS validation failure, reproducible*. Some results returned by the API include properties not defined in the API specification, namely: `response._links`, `response.liked_posts[].blog`, `response.liked_posts[].id_string`. **Test ID**: `test_1h7fu18iwnmeo_getBlogLikes`.

### Yelp - Search bussinesses
  - *500 status code, reproducible*. The parameter `open_at` accepts a timestamp representing a future date. When sending a value higher than the maximum int32 value such as `2147483653`, the API returns a server error. **Test ID**: `test_1hrgf37q8oca8_getBusinesses`.
  - *500 status code, non-reproducible*. Multiple requests (including valid and faulty ones) obtained server errors. **Test ID**: `test_1hbd8xv6txw6w_getBusinesses`.
  - *Error response after valid request, reproducible*. When setting the `location` parameter to `Egypt` and the `language` to `fi_FI` (Finnish), the error `LOCATION_NOT_FOUND` is returned. However, changing the language makes the error disappear and actual results are returned. **Test ID**: `test_r1cuuplspdnq_getBusinesses`.
  - *Successful response after request violating dependency, reproducible*. Requests using the parameters `open_now` and `open_at` set to `false` obtain successful responses, even though the API documentation explicitly states that both parameters cannot be used together. **Test ID**: `test_1h7wh51d7ps1i_getBusinesses`.
  - *OAS validation failure, reproducible*. For some results returned by the API, the enum property `businesses[].price` includes values not enumerated in the documentation ([`$`, `$$`, `$$$`, `$$$$`]), for example, `€` and `￥￥￥`. **Test ID**: `test_trz3ptplfcj6_getBusinesses`.
  - *OAS validation failure, reproducible*. One result returned by the API has its `businesses[].rating` propoerty set to `0.0`, even though the documentation explicitly states that this value must be between 1 and 5. **Test ID**: `test_1iusr05s43p0l_getBusinesses`.

### Yelp - Search transactions
  - *500 status code, non-reproducible*. Multiple requests obtained server errors. **Test ID**: `test_1hrdpkzrwyjj7_getTransactions`.
  - *Error response after valid request, reproducible*. When setting the `location` parameter to `Daca, Bangladesh`, the error `VALIDATION_ERROR` is returned. This is a valid location, therefore an error should not be returned. **Test ID**: `test_1h82k6ahbkb3s_getTransactions`.

### YouTube - Get comment threads
  - *Error response after valid request, non-reproducible*. A valid request obtained a client error categorised as "processingFailure", with the following message: `The API server failed to successfully process the request. While this can be a transient error, it usually indicates that the requests input is invalid. Check the structure of the <code>commentThread</code> resource in the request body to ensure that it is valid.`. Such message makes no sense, since this is a read operation and no body was included in the request. **Test ID**: `test_qhiay3nt7t4j_youtubecommentThreadslist`.
  - *Successful response after request violating dependency, reproducible*. Requests using the parameters `id` and `order` obtain successful responses, even though the API documentation explicitly states that the `order` parameter is not supported for use in conjunction with the `id` parameter. **Test ID**: `test_1h7j4wavuba79_youtubecommentThreadslist`.
  - *OAS validation failure, reproducible*. For some results returned by the API, the enum property `items[].snippet.topLevelComment.snippet.moderationStatus` is set to `likelyspam`, which is a value not enumerated in the documentation ([`heldForReview`, `likelySpam`, `published`, `rejected`]).

### YouTube - Get videos
  - *Successful response after request violating dependency, reproducible*. Requests using the parameter `regionCode` without also setting `chart` obtain successful responses, even though the API documentation explicitly states that the `regionCode` parameter can only be used in conjunction with the `chart` parameter. **Test ID**: `test_1h7iolpche2ch_youtubevideoslist`.
  - *Successful response after request violating dependency, reproducible*. Requests using the parameters `maxResults` and `id` obtain successful responses, even though the API documentation explicitly states that the `maxResults` parameter is not supported for use in conjunction with the `id` parameter. **Test ID**: `test_1h821gldox7w0_youtubevideoslist`.
  - *Successful response after request violating dependency, reproducible*. Requests using the parameter `videoCategoryId` without also setting `chart` obtain successful responses, even though the API documentation explicitly states that the `videoCategoryId` parameter can only be used in conjunction with the `chart` parameter. **Test ID**: `test_1hbgh2v8tubed_youtubevideoslist`.
  - *Successful response after request with faulty parameter, non-reproducible*. In two occassions, requests ommitting the mandatory parameter `part` obtained successful responses. **Test ID**: `test_s0xb0oqbliyg_youtubevideoslist`.

### YouTube - Search
  - *Error response after valid request, reproducible*. There exist two unspecified dependencies in this service. They cause one of every two supposedly valid requests to be rejected.  **Test ID**: `test_1h7fu0woe00tv_youtubesearchlist`.
  - *OAS validation failure, reproducible*. Some results returned by the API include the property `items[].snippet.publishTime`, which is not defined in the API specification. **Test ID**: `test_1hbghsasm15cz_youtubesearchlist`.
