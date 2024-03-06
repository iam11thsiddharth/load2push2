# Load2Push

Load2Push is a service that allows users to insert data into a Baserow table using URL parameters. Follow the steps below to get started.

## Prerequisites

1. **Create a Baserow Account**: If you don't already have one, sign up for a [Baserow](https://baserow.io/) account.

2. **Create a Baserow Table**: After logging in, create a new table in Baserow. Add columns to your table corresponding to the data you want to insert. Ensure all column names are in lowercase.

3. **Generate API Keys**: Go to your Baserow account settings and generate API keys. You'll need these keys to authenticate your requests to the Baserow API.

## Getting Started

1. **Sign Up for Load2Push**: Visit [Load2Push](https://load2push.netlify.app/) and sign up for an account using your email address.

2. **Enter Baserow API Keys**: Once logged in, navigate to your account settings and enter your Baserow API keys. These keys will be securely stored and used to interact with your Baserow tables.

3. **Get ID for Your Table**: After entering your API keys, you'll receive an ID for your table. This ID will be used in the URL to specify the table where data should be inserted.

## Usage

To insert data into your Baserow table, follow these steps:

1. **Construct the URL**: Use the following format to construct your URL:

https://load2push.worker.dev/?id=[YOUR_TABLE_ID]&[COLUMN_NAME_1]=[VALUE_1]&[COLUMN_NAME_2]=[VALUE_2]&...

Replace `[YOUR_TABLE_ID]` with the ID provided after entering your API keys. Replace `[COLUMN_NAME_1]`, `[COLUMN_NAME_2]`, etc., with the names of the columns you created in your Baserow table (all lowercase). Replace `[VALUE_1]`, `[VALUE_2]`, etc., with the values you want to insert into each column.

2. **Load the URL**: Open the constructed URL in your browser or send a request to it programmatically. This will trigger Load2Push to insert the data into your Baserow table.

## Example

Suppose you have a Baserow table with the following columns:

- `name`
- `class`
- `age`

And you want to insert the following data:

- name: John
- class: 2
- Age: 30

Your constructed URL would look like this:

https://load2push.worker.dev/?id=[YOUR_ID provided in dashboard]&name=John&class=2&age=30


## IP Address

You can use the `ip=yes` parameter in the URL to store the IP address of the user in your Baserow table if a column named `ip` exists.

## Notes

- Ensure that the column names in the URL parameters match exactly with the column names in your Baserow table.
- Load2Push can only insert data into Baserow tables; it does not have the capability to fetch data from Baserow.



**Note:**

- This Cloudflare Worker script can only insert data into a Baserow table. It does not have the capability to fetch data from Baserow.
- You can use `ip=yes` parameter to store the IP address of the user in your table (a column named - `ip` must exist).
- Parameter names must be exactly the same as your Baserow table's column names.
- Avoid using capital alphabets in your database table/column/row or parameters and others.

**Cloudflare worker code deployed at load2push.worker.dev :**
  
  
  ```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  try {
    // Extract URL parameters
    const url = new URL(request.url)
    const params = url.searchParams
    const id = params.get('id')
    if (!id) {
      return new Response('Error: Missing id parameter.', { status: 400 })
    }

    // Baserow API endpoint to fetch full row data
    const baserowApiRowUrl = `https://api.baserow.io/api/database/rows/table/263653/${id}/?user_field_names=true`

    // Fetch full row data from Baserow API
    const responseRow = await fetch(baserowApiRowUrl, {
      headers: {
        'Authorization': `Token [Your Baserow API Key]`, // Replace with your actual API key
      },
    })

    if (!responseRow.ok) {
      const errorText = `Failed to fetch row data from Baserow. Status: ${responseRow.status}`
      console.error(errorText)
      return new Response(errorText, { status: responseRow.status })
    }

    // Extract row data from the response
    const rowData = await responseRow.json()
    console.log('Row data:', rowData)

    // Extract auth and table values from the row data
    const auth = rowData.auth
    const table = rowData.table

    if (!auth || !table) {
      const errorText = 'Unable to retrieve authentication token or table ID from Baserow.'
      console.error(errorText)
      return new Response(errorText, { status: 500 })
    }

    console.log('Authentication token:', auth)
    console.log('Table ID:', table)

    // Prepare data object to store URL parameters
    const data = {}
    
    // Iterate over URL parameters and add them to the data object
    params.forEach((value, key) => {
      // Check if parameter contains "ip"="yes"
      if (key === 'ip' && value === 'yes') {
        // Fetch current user IP address
        const ipAddress = request.headers.get('CF-Connecting-IP') || request.headers.get('X-Forwarded-For') || request.headers.get('Remote-Addr') || request.headers.get('X-Real-IP') || request.headers.get('CF-IPCountry')
        // Add IP address to data object
        data['ip'] = ipAddress
      } else {
        // Add other parameters to data object
        data[key] = value
      }
    })

    // Baserow API endpoint to add data to the specified table
    const baserowApiUrl = `https://api.baserow.io/api/database/rows/table/${table}/?user_field_names=true`

    // Make a POST request to Baserow API to add data to the specified table
    const response = await fetch(baserowApiUrl, {
      method: 'POST',
      headers: {
        'Authorization': `Token ${auth}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    })

    if (!response.ok) {
      const errorText = `Failed to add data to Baserow. Status: ${response.status}`
      console.error(errorText)
      console.error('Response:', response)
      const responseText = await response.text()
      console.error('Response text:', responseText)
      return new Response(errorText, { status: response.status })
    }

    return new Response('Data added to Baserow successfully.', { status: 200 })
  } catch (error) {
    console.error('Error:', error)
    return new Response('Internal Server Error.', { status: 500 })
  }
}
```





##### Credits

developed by [Siddharth](https://github.com/iam11thsiddharth).


