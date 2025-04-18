# Regression Suite Flow 


- [UserSignup](#UserSignup)
- [UserLogin](#UserLogin)
- [GenerateTokens](#GenerateTokens)

 
 
## **UserSignup**
- Generates test users using `load_test_users()`.
- Attempts to **sign up each user** through the `/signup` endpoint.
- Verifies that at least **10 users are successfully signed up**.
- Logs responses for debugging purposes.
- Stores successfully signed-up users in the `successful_users` list.


## **UserLogin**
- Takes **5 valid users** (from `successful_users`) and attempts valid logins through the `/login` endpoint.
- Takes **5 invalid users** (from `load_invalid_users()`) to test invalid logins.
- Verifies:
  - **5 successful logins** for valid users.
  - **5 failed logins** for invalid users.
- Logs all login responses for debugging.


## **GenerateTokens**
- Logs in **5 valid users** (can be modified to 5) and retrieves their `access_token` and `refresh_token`.
- Tests **refresh token flow**:
  - **Before token expiry:** Tries to refresh the token (expects failure with `325` status code).
  - **After token expiry:** Waits for the `access_token` to expire, then refreshes the token (expects success with `200` status code).
- Tests **invalid scenarios**:
  - Refreshing with an invalid refresh token (expects failure with `417` status code).
  - Refreshing without providing a refresh token (expects failure with `400` status code).

