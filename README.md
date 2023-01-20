# Lambda Argon
Utilize the Argon2ID cryptographically secure hashing algorithm with parameters tuned specifically to run optimally on AWS Lambda.

Originally forked from github.com/alexedwards/argon2id but modified significantly to work optimally with AWS Lambda, cleanup test cases, and add new test cases.

# Original README
This package provides a convenience wrapper around Go's [argon2](https://pkg.go.dev/golang.org/x/crypto/argon2?tab=doc) implementation, making it simpler to securely hash and verify passwords using Argon2.
It enforces use of the Argon2id algorithm variant and cryptographically-secure random salts.

## How to Build locally
1. `make build`

## How to Test locally
1. `make test`

## How to Use
1. `go get github.com/seantcanavan/lambda_argon@latest`
2. `import github.com/seantcanavan/lambda_argon`
3. Steps for adding or updating a password for a user:
   1. Get the user password input from your lambda req: `req.Body.password` or `req.QueryStringParameters["password"]` or similar
   2. Hash the password to store in your database: `hash, err := lambda_argon.Hash(password)`
   3. Store the hash in your database: `user.UpdatePassword(ctx, hash)`
4. Steps for validating passwords / logging in for a user:
   1. Get the user password input from your lambda req: `req.Body.password` or `req.QueryStringParameters["password"]` or similar
   2. Get the hash for the user in your database: `hash, err := user.GetHash(ctx, userID)`
   3. Try to match the user input against the hash: `match, err := lambda_argon.Match(password, hash)`

## Sample Login Lambda Handler Example
``` go
func LoginLambda(ctx context.Context, lambdaReq events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	var loginReq LoginReq
	err := json.Unmarshal(lambdaReq.Body, &loginReq)
	if err != nil {
		return ERROR - internal server
	}

	adminByEmail, err := admin.GetByEmail(ctx, loginReq.Email)
	if err != nil {
		return ERROR - not found
	}

	// check if the password provided matches the hashed one saved in this admin
	match, err := lambda_argon.Match(loginReq.Password, adminByEmail.Password)
	if err != nil {
		return ERROR - bad request
	}

	// check if the password matches
	if !match {
		return ERROR - unauthorized
	}

	return SUCCESS
}
```

## Sample Update Password Lambda Handler Example
``` go
func UpdatePasswordLambda(ctx context.Context, lambdaReq events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	var updatePasswordReq UpdatePasswordReq
	err := json.Unmarshal(lambdaReq.Body, &updatePasswordReq)
	if err != nil {
		return ERROR - internal server
	}

    adminByEmail, err := admin.GetByEmail(ctx, loginReq.Email)
	if err != nil {
		return ERROR - not found
	}

	match, err := lambda_argon.Match(updatePasswordReq.Password, adminByEmail.Password)
	if err != nil {
		return ERROR - bad request
	}

	// check if the password matches
	if !match {
		return ERROR - unauthorized
	}

	hash, err := lambda_argon.Hash(updatePasswordReq.NewPassword)
	if err != nil {
		return ERROR - bad request
	}

	httpStatus, err = admin.SetPassword(ctx, adminById.ID, hash)
	if err != nil {
		return ERROR - conflict
	}

    return SUCCESS
}
```

## All tests are passing
```
Fri Jan 20 03:33 PM lambda_argon: make test
go test ./... -v
=== RUN   TestCreateHash
=== RUN   TestCreateHash/verify_hash_output_matches_expected_regex_for_short_password
=== RUN   TestCreateHash/verify_same_password_produces_different_hashes_for_short_password
=== RUN   TestCreateHash/verify_hash_output_matches_expected_regex_for_long_password
=== RUN   TestCreateHash/verify_same_password_produces_different_hashes_for_long_password
--- PASS: TestCreateHash (0.73s)
    --- PASS: TestCreateHash/verify_hash_output_matches_expected_regex_for_short_password (0.00s)
    --- PASS: TestCreateHash/verify_same_password_produces_different_hashes_for_short_password (0.00s)
    --- PASS: TestCreateHash/verify_hash_output_matches_expected_regex_for_long_password (0.00s)
    --- PASS: TestCreateHash/verify_same_password_produces_different_hashes_for_long_password (0.00s)
=== RUN   TestCheckHash
=== RUN   TestCheckHash/verify_checkHash_works_for_short_password_/_hash
=== RUN   TestCheckHash/verify_checkHash_works_for_long_password_/_hash
=== RUN   TestCheckHash/verify_checkHash_with_hard_coded_values
=== RUN   TestCheckHash/verify_checkHash_fails_with_tampered_hash_value
=== RUN   TestCheckHash/verify_err_when_using_wrong_argon_variant
--- PASS: TestCheckHash (0.73s)
    --- PASS: TestCheckHash/verify_checkHash_works_for_short_password_/_hash (0.18s)
    --- PASS: TestCheckHash/verify_checkHash_works_for_long_password_/_hash (0.18s)
    --- PASS: TestCheckHash/verify_checkHash_with_hard_coded_values (0.02s)
    --- PASS: TestCheckHash/verify_checkHash_fails_with_tampered_hash_value (0.00s)
    --- PASS: TestCheckHash/verify_err_when_using_wrong_argon_variant (0.00s)
=== RUN   TestComparePasswordAndHash
=== RUN   TestComparePasswordAndHash/verify_compare_with_correct_short_password
=== RUN   TestComparePasswordAndHash/verify_err_with_wrong_password_for_short_password_hash
=== RUN   TestComparePasswordAndHash/verify_compare_with_correct_long_password
=== RUN   TestComparePasswordAndHash/verify_err_with_wrong_password_for_long_password
--- PASS: TestComparePasswordAndHash (1.09s)
    --- PASS: TestComparePasswordAndHash/verify_compare_with_correct_short_password (0.18s)
    --- PASS: TestComparePasswordAndHash/verify_err_with_wrong_password_for_short_password_hash (0.20s)
    --- PASS: TestComparePasswordAndHash/verify_compare_with_correct_long_password (0.18s)
    --- PASS: TestComparePasswordAndHash/verify_err_with_wrong_password_for_long_password (0.18s)
=== RUN   TestDecodeHash
=== RUN   TestDecodeHash/verify_shortParams_returned_are_correct
=== RUN   TestDecodeHash/verify_shortSalt_length_is_correct
=== RUN   TestDecodeHash/verify_shortKey_length_is_correct
=== RUN   TestDecodeHash/verify_longParams_returned_are_correct
=== RUN   TestDecodeHash/verify_longSalt_length_is_correct
=== RUN   TestDecodeHash/verify_longKey_length_is_correct
--- PASS: TestDecodeHash (0.35s)
    --- PASS: TestDecodeHash/verify_shortParams_returned_are_correct (0.00s)
    --- PASS: TestDecodeHash/verify_shortSalt_length_is_correct (0.00s)
    --- PASS: TestDecodeHash/verify_shortKey_length_is_correct (0.00s)
    --- PASS: TestDecodeHash/verify_longParams_returned_are_correct (0.00s)
    --- PASS: TestDecodeHash/verify_longSalt_length_is_correct (0.00s)
    --- PASS: TestDecodeHash/verify_longKey_length_is_correct (0.00s)
=== RUN   TestHash
=== RUN   TestHash/verify_password_hashes_without_error
--- PASS: TestHash (0.17s)
    --- PASS: TestHash/verify_password_hashes_without_error (0.17s)
=== RUN   TestMatch
=== RUN   TestMatch/verify_password_matches_without_error
--- PASS: TestMatch (0.36s)
    --- PASS: TestMatch/verify_password_matches_without_error (0.36s)
=== RUN   TestVerifyPasswordRequirements
=== RUN   TestVerifyPasswordRequirements/verify_error_when_password_is_below_minimum
=== RUN   TestVerifyPasswordRequirements/verify_error_when_password_is_above_maximum
=== RUN   TestVerifyPasswordRequirements/verify_no_error_when_input_is_minimum
=== RUN   TestVerifyPasswordRequirements/verify_no_error_when_input_is_maximum
--- PASS: TestVerifyPasswordRequirements (0.00s)
    --- PASS: TestVerifyPasswordRequirements/verify_error_when_password_is_below_minimum (0.00s)
    --- PASS: TestVerifyPasswordRequirements/verify_error_when_password_is_above_maximum (0.00s)
    --- PASS: TestVerifyPasswordRequirements/verify_no_error_when_input_is_minimum (0.00s)
    --- PASS: TestVerifyPasswordRequirements/verify_no_error_when_input_is_maximum (0.00s)
PASS
ok  	github.com/seantcanavan/lambda_argon	3.437s
```
