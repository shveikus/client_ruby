# beGateway Ruby client

The gem provides easy way to connect and send requests to beGateway payment platform.

## Installation

Add this line to your application's Gemfile:

    gem 'be_gateway', '~> 0.12'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install be_gateway

## Usage

### Intialization client

``` ruby
client = BeGateway::Client.new({
  shop_id: 'YOUR SHOP ID',
  secret_key: 'YOUR SHOP SECRET KEY',
  url: 'YOUR GATEWAY URL'
})
```

### Available methods:

  * authorize
  * capture
  * void
  * payment
  * refund
  * credit
  * checkup
  * query
  * close_days
  * v2_create_card
  * v2_get_card_by_token


**Pay attention** that client add main **'request'** section automatically, and you don't need to describe it

### Transaction Authorization example

The request used to verify cardholder's funds. It is typically employed when merchants do not fulfill orders immediately.

``` ruby
response = client.authorize({
  amount: 100,
  currency: 'USD',
  description: 'Test transaction',
  tracking_id: 'tracking_id_000',
  billing_address: {
    first_name: 'John',
    last_name: 'Doe',
    country: 'US',
    city: 'Denver',
    state: 'CO',
    zip: '96002',
    address: '1st Street'
  },

  credit_card: {
    number: '4200000000000000',
    verification_value: '123',
    holder: 'John Doe',
    exp_month: '05',
    exp_year: '2020',
  },

  customer: {
    ip: '127.0.0.1',
    email: 'john@example.com'
  }
})

response.transaction.id # => returns id of processed transaciton
response.transaction.status # => returns status of processed transaciton

response.authorization.auth_code
response.authorization.rrn
```

### Transaction Payment

Payment transaction is a combination of authorization and capture processed at a time. This transaction type is generally used when the goods or services can be immediately provided to the customer.

``` ruby
response = client.payment(params)
```
Where `params` have same structure as **Authorization**

### Transaction Checkup

This transaction allow you to check transaction via **beProtected** system.

``` ruby
response = client.checkup(params)
```
Where `params` have same structure as **Authorization**

### Transaction Refund example

The refund allows to credit the customer, e.g. in case of returned goods or cancelation. To post a refund request, a valid transaction id from a former Capture or Payment transaction is required. It is only possible to credit an amount less than or equal to the initial transaction using the same currency as with the original transaction. This feature also allows you to issue multiple partial refunds against an original transaction.

``` ruby
response = client.refund({
  parent_uid:  'UID of original Payment or Capture transactions',
  amount:      'Amount of refund',
  reason:      'Reason of refund. Ex "Client request"'
})

response.transaction.uid    # => returns uid of processed transaciton
response.transaction.status # => returns status of processed transaciton
```

### Transaction Capture

After an order is shipped, a previous authorized amount can be settled (captured). The card issuing bank credits the funds to the merchant's bank account and updates the cardholder's statement. Card regulations require a merchant to ship goods before settling the funds for an order.

``` ruby
response = client.capture(params)
```
Where `params` have same structure as **Refund**

### Transaction Void

The request allows you to void a transaction that has been previously authorized and is still pending settlement. Voiding a transaction cancels the authorization process and prevents the transaction from being submitted to the processor for settlement.

``` ruby
response = client.void(params)
```
Where `params` have same structure as **Refund**

### Transaction Credit example

The request credits (pushes) funds to a recipient's card account. The transaction is not supported by all acquiring banks and the transaction is not available to all merchants.

``` ruby
response = client.credit({
  amount: 100,
  currency: "USD",
  description: "Test transaction",
  tracking_id: "tracking_id_000",
  credit_card: {
    token: "Token from successful Payment/Authorization transaction"
  }
})

response.transaction.uid    # => returns uid of processed transaciton
response.transaction.status # => returns status of processed transaciton
```

### Query Request example

Quickly look up API transaction results.

``` ruby
response = client.query(id: transaction_id)

# or you can get transaction by tracking_id

response = client.query(tracking_id: 'your tracking id')

response.transaction.id # => returns id of processed transaciton
response.transaction.status # => returns status of processed transaciton
```

### Close day

The request manually close a business day on a merchant terminal and activates funds clearing to your bank account. The feature is not supported by all acquiring institutions. You must know your gateway id you want to close the business day. The gateway id exists in `authorization` or `payment` transaction responses.

```ruby
response = client.close_days(gateway_id: 1)

response.transaction.status # => returns status of processed transaciton
```

### Tokenize card

The request allows you to tokenize credit card to use returned token further in `authorization` or `payment` transactions instead of full card number.

```ruby
response = client.v2_create_card({
  'number' => 4200000000000000,
  'holder' => "John Smith",
  'exp_month' => 05,
  'exp_year' => 2030,
  'public_key' => 'your_shop_public_key'
})

response.token # => returns card token
```

### Get card details by token

The request gets tokenized card details.

```ruby
response = client.v2_get_card_by_token('token')

response.holder # => returns card holder
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
