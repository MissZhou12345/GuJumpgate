## ADDED Requirements
### Requirement: Hosted Checkout Send-Link Fetch
The side panel SHALL provide a send-link URL field above the Hosted Checkout verification URL row, with an input and a fetch button for obtaining PayPal SMS configuration.

#### Scenario: URL is missing
- **WHEN** the user clicks the fetch button with an empty send-link URL
- **THEN** the side panel SHALL show a validation message
- **AND** it SHALL NOT request any URL

#### Scenario: URL is invalid
- **WHEN** the user clicks the fetch button with a malformed send-link URL
- **THEN** the side panel SHALL show a validation message
- **AND** it SHALL NOT request any URL

#### Scenario: Send-link request returns detail error
- **WHEN** the send-link request returns a response containing `detail` without a valid `code_url` and `phone`
- **THEN** the side panel SHALL show the `detail` value to the user
- **AND** it SHALL NOT update the Hosted Checkout verification URL
- **AND** it SHALL NOT update the PayPal phone field
- **AND** the user SHALL be able to click the fetch button again

#### Scenario: Send-link request succeeds
- **WHEN** the send-link request returns valid `code_url`, `ready_url`, `resend_url`, `complete_url`, and `phone`
- **THEN** the side panel SHALL set the Hosted Checkout verification URL to `code_url`
- **AND** it SHALL set the PayPal phone field to `phone` without a leading US `+1` or Japan `+81` country code
- **AND** it SHALL preserve `ready_url`, `resend_url`, and `complete_url` for later use
- **AND** it SHALL sync the updated Hosted Checkout settings through the existing profile/state flow.
