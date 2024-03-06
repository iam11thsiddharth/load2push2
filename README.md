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

##### Credits

This Cloudflare Worker script was developed by [Siddharth](https://github.com/iam11thsiddharth).


