import pandas as pd
import requests
import concurrent.futures
import time

def make_request(url, request_type="GET", payload=None, config=None):
    start_time = time.time()
    try:
        if request_type == "GET":
            response = requests.get(url, params=payload, **config)
        elif request_type == "POST":
            response = requests.post(url, data=payload, **config)
        # Add more elif conditions for other HTTP methods if needed

        end_time = time.time()
        elapsed_time = end_time - start_time
        return response.status_code, elapsed_time
    except Exception as e:
        return str(e), 0

def make_concurrent_requests(endpoint, request_type="GET", payload=None, config=None, n=10):
    urls = [endpoint] * n
    total_elapsed_time = 0
    successful_responses = 0
    unsuccessful_responses = 0

    with concurrent.futures.ThreadPoolExecutor() as executor:
        results = list(executor.map(make_request, urls, [request_type]*n, [payload]*n, [config]*n))

    for result in results:
        status_code, elapsed_time = result
        total_elapsed_time += elapsed_time
        if isinstance(status_code, int) and status_code == 200:
            successful_responses += 1
        else:
            unsuccessful_responses += 1

    average_time = total_elapsed_time / n if n > 0 else 0

    return average_time, successful_responses, unsuccessful_responses

def process_excel(input_file, output_file):
    # Read data from Excel file
    df = pd.read_excel(input_file)

    # Iterate over rows
    for index, row in df.iterrows():
        endpoint = row['Endpoint/URL']
        payload = row['Payload']
        request_type = row['Request Type']
        n = row['Number of Requests']  # Number of requests for this row

        # Execute script
        average_time, successful_responses, unsuccessful_responses = make_concurrent_requests(endpoint, request_type=request_type, payload=payload, n=n)

        # Update DataFrame with metrics
        df.at[index, 'Average Time'] = average_time
        df.at[index, 'Successful Requests'] = successful_responses
        df.at[index, 'Unsuccessful Requests'] = unsuccessful_responses

    # Save updated DataFrame to Excel file
    df.to_excel(output_file, index=False)

if __name__ == "__main__":
    input_file = "input_data.xlsx"
    output_file = "output_data.xlsx"

    process_excel(input_file, output_file)
