# Change: Add Hosted Checkout send-link fetch field

## Why
PayPal Hosted Checkout 接码配置目前需要用户手动填写 PayPal 电话和验证码接口。用户希望通过一个发码链接接口自动获取这些配置，减少手动复制并保留后续会用到的状态接口。

## What Changes
- 在 `row-hosted-checkout-verification-url` 上方新增“发码链接”行，包含 URL 输入框和“获取”按钮。
- 点击“获取”时校验 URL 必填且格式有效，然后请求该 URL。
- 请求未正确返回时，将响应中的 `detail` 提示给用户，不更新 PayPal 电话、验证码接口或其他状态，按钮可继续点击。
- 请求正确返回时，将 `code_url` 写入验证码接口，将 `phone` 写入 PayPal 电话并去掉美区 `+1` 或日区 `+81` 前缀。
- 暂存 `ready_url`、`resend_url`、`complete_url`，供后续流程使用。

## Impact
- Affected specs: `hosted-checkout-sms`
- Affected code: `sidepanel/sidepanel.html`, `sidepanel/sidepanel.js`, likely `background.js` persisted settings if the temporary URLs need to survive panel reloads.
