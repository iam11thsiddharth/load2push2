# Load2Push

This Cloudflare Worker script interacts with the Baserow API to insert data into a specified table. It does not have the capability to fetch data from Baserow.

## Usage

1. **Clone this repository:**

    ```bash
    git clone https://github.com/iam11thsiddharth/load2push.git
    ```

2. **Install dependencies (none required for Cloudflare Workers).**



4. **Configure the necessary environment variables in the application at [load2push](https://load2push.netlify.app/).:**

   - `BASEROW_API_KEY`: Your Baserow API key.
   - `BASEROW_TABLE_ID`: The ID of the Baserow table where data will be inserted.
   - `BASEROW_ROW_ID`: The ID of the Baserow row to fetch data from.

5. **Use the Cloudflare Worker URL in your applications to insert data into Baserow.**

### Features

- Inserts data into a specified table in Baserow using POST requests.
- Optionally fetches the current user's IP address and sends it to a designated column in the Baserow table if requested.

#### Example Usage

- To insert data into Baserow:

    ```plaintext
    https://load2push.worker.dev/?id=123&param1=value1&param2=value2
    ```

- To fetch the current user's IP and insert it into Baserow:

    ```plaintext
    https://load2push.worker.dev/?id=123&ip=yes&param1=value1&param2=value2
    ```

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


