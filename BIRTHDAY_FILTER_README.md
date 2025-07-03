# Birthday Filter Feature for Savings Accounts API

## Overview

This feature adds birthday filtering capabilities to the `/api/v1/savingsaccounts` endpoint in Apache Fineract. It allows you to filter savings accounts by the client's birthday (month and day), which is useful for building birthday dashboards and eventually automating birthday emails.

## Implementation Details

### Files Modified

1. **`fineract-provider/src/main/java/org/apache/fineract/portfolio/savings/api/SavingsAccountsApiResource.java`**
   - Added `birthdayMonth` and `birthdayDay` query parameters to the `retrieveAll` method
   - Updated API documentation to include example usage

2. **`fineract-core/src/main/java/org/apache/fineract/infrastructure/core/service/SearchParameters.java`**
   - Added `birthdayMonth` and `birthdayDay` fields
   - Updated `forSavings` method to accept birthday parameters
   - Added getter methods for the new fields

3. **`fineract-provider/src/main/java/org/apache/fineract/portfolio/savings/service/SavingsAccountReadPlatformServiceImpl.java`**
   - Added birthday filtering logic in the `retrieveAll` method
   - Uses SQL `EXTRACT` function to filter by month and day of birth from the `m_client.date_of_birth` column

## API Usage

### Endpoint
```
GET /fineract-provider/api/v1/savingsaccounts
```

### Query Parameters

- `birthdayMonth` (optional): Integer representing the month (1-12)
- `birthdayDay` (optional): Integer representing the day (1-31)
- `tenantIdentifier` (required): Tenant identifier (e.g., "default")

### Examples

#### Get all savings accounts for clients with birthday on December 8th:
```bash
curl -u mifos:password -k -X GET \
  "https://localhost:8443/fineract-provider/api/v1/savingsaccounts?birthdayMonth=12&birthdayDay=8&tenantIdentifier=default"
```

#### Get all savings accounts for clients with birthday on January 15th:
```bash
curl -u mifos:password -k -X GET \
  "https://localhost:8443/fineract-provider/api/v1/savingsaccounts?birthdayMonth=1&birthdayDay=15&tenantIdentifier=default"
```

#### Get all savings accounts (no birthday filter):
```bash
curl -u mifos:password -k -X GET \
  "https://localhost:8443/fineract-provider/api/v1/savingsaccounts?tenantIdentifier=default"
```

## Response Format

The response format remains the same as the existing endpoint:

```json
{
  "totalFilteredRecords": 5,
  "pageItems": [
    {
      "id": 1,
      "accountNo": "000000001",
      "depositType": {
        "id": 200,
        "code": "depositAccountType.fixedDeposit",
        "value": "Fixed Deposit"
      },
      "clientId": 1,
      "clientName": "Jane Saver",
      "savingsProductId": 2,
      "savingsProductName": "birthday-bank-savings-product"
    }
  ]
}
```

## Database Query

The birthday filtering uses the following SQL logic:
```sql
EXTRACT(MONTH FROM c.date_of_birth) = ? AND EXTRACT(DAY FROM c.date_of_birth) = ?
```

This extracts the month and day from the client's `date_of_birth` field in the `m_client` table and filters accordingly.

## Testing

The feature includes integration tests in `integration-tests/src/test/java/org/apache/fineract/integrationtests/SavingsAccountsTest.java` to verify the functionality.

## Notes

- Both `birthdayMonth` and `birthdayDay` must be provided together for the filter to work
- If only one parameter is provided, the filter is ignored
- The filter works with the existing pagination and other query parameters
- The feature is backward compatible - existing API calls will continue to work without modification 