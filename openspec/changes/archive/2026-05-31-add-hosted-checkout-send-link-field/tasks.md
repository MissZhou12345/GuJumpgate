## 1. Implementation
- [x] 1.1 Add persisted/profile fields for the send-link URL and returned `ready_url`, `resend_url`, `complete_url` if they must survive side panel refresh.
- [x] 1.2 Add the send-link input row above `row-hosted-checkout-verification-url`.
- [x] 1.3 Implement URL validation, fetch handling, error `detail` display, and retry-safe button state.
- [x] 1.4 On successful response, set the hosted checkout verification URL and normalized PayPal phone number, then sync the active Plus checkout profile/state.
- [x] 1.5 Preserve returned `ready_url`, `resend_url`, and `complete_url` for later use.
- [ ] 1.6 Run `node --check sidepanel/sidepanel.js` and any touched JavaScript files.
## 2. Manual Verification
- [ ] 2.1 Verify empty/invalid send-link URL shows a user-facing validation error.
- [ ] 2.2 Verify `{ "detail": "..." }` response displays the detail without changing existing PayPal phone or verification URL.
- [ ] 2.3 Verify a success response fills `code_url` into 验证码接口 and strips `+1` / `+81` from PayPal 电话.
