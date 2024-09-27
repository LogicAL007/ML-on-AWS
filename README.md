### PTwitter Sentiment Analysis with AWS Lambda

This project demonstrates a serverless application that pulls recent tweets from Twitter, analyzes their sentiment, and stores the results in both an AWS S3 bucket and a PostgreSQL database. The main components include AWS Lambda, Twitter API integration, sentiment analysis using NLTK, and the interaction with AWS S3 and a relational database.

### Features:
1. **Tweet Retrieval**:
   - Uses the Twitter API (via `Twython`) to retrieve recent tweets from a specific user (Reuters in this case).
   - Filters tweets to get only the recent ones (based on the `created_at` timestamp).

2. **Sentiment Analysis**:
   - Sentiment analysis is performed on the tweet text using NLTK's `SentimentIntensityAnalyzer` from the VADER lexicon, providing a sentiment score between -1 (negative) and 1 (positive).

3. **Data Storage**:
   - The processed tweets are saved as JSON files in an AWS S3 bucket.
   - The tweet data (with sentiment scores) is also inserted into a PostgreSQL database for further analysis or reporting.

### Code Breakdown:

#### `lambda_handler(event, context)`:
This is the main entry point for the AWS Lambda function. It orchestrates the overall workflow:
1. **Twitter API Connection**: Uses API keys stored in environment variables to authenticate and fetch the timeline of a specified Twitter user (Reuters by default).
2. **Filter Tweets**: Filters tweets that are posted within the last 5 minutes.
3. **Sentiment Analysis**: Analyzes the sentiment of each tweet's text using VADER.
4. **Save Tweets to S3**: Serializes the tweets (converting timestamps to Unix format) and uploads them to an S3 bucket as JSON files.
5. **Store Tweets in Database**: Inserts the tweet data along with the sentiment score into a PostgreSQL database table.

#### Supporting Functions:
- **`_time_parser(twitter_time: str)`**: Converts Twitter's timestamp string into a Python `datetime` object.
- **`is_recent(tweet: dict)`**: Checks if a tweet is recent by comparing its creation time with the current time.
- **`extract_fields(tweet: dict)`**: Extracts essential fields (author, timestamp, and text) from the tweet for further processing.
- **`_get_sentiment(string: str)`**: Calculates the sentiment score of a tweet's text using VADER sentiment analysis.
- **`add_sentiment_score(tweet: dict)`**: Adds the sentiment score to the tweet dictionary.
- **`upload_file_to_s3(local_file_name: str, bucket: str)`**: Uploads the local file (tweets JSON) to an AWS S3 bucket.
- **`get_db_connection()`**: Establishes a connection to a PostgreSQL database using credentials stored in environment variables.
- **`convert_timestamp_to_int(tweet: dict)`**: Converts a `datetime` object to a Unix timestamp for JSON serialization.
- **`insert_data_in_db(df: pd.DataFrame, conn: psycopg2.extensions.connection)`**: Inserts the tweet data (as a pandas DataFrame) into the specified PostgreSQL table.

### Installation and Setup:

1. **Twitter API Keys**:
   - You must have a valid Twitter Developer account and API keys (API Key and API Secret Key).
   
2. **AWS Setup**:
   - **AWS Lambda**: Set up an AWS Lambda function with an appropriate role that grants access to S3 and logs.
   - **S3 Bucket**: Create an S3 bucket to store the tweet data as JSON files.
   - **PostgreSQL Database**: You need an accessible PostgreSQL instance (hosted on AWS RDS or another service) and credentials stored as environment variables.

3. **Environment Variables**:
   The following environment variables should be configured for Lambda:
   - `TWITTER_API_KEY`: Twitter API Key.
   - `TWITTER_API_SECRET`: Twitter API Secret Key.
   - `S3_BUCKET_NAME`: Name of the AWS S3 bucket.
   - `DB_HOST`: PostgreSQL database host.
   - `DB_PASSWORD`: PostgreSQL database password.

### Usage:
1. **Deploy to AWS Lambda**: Package the code and deploy it as an AWS Lambda function. Set up a trigger for this Lambda (e.g., on a schedule or through an event).
2. **Monitor Execution**: Check AWS CloudWatch logs for the status of the function and any potential errors.

### Example Workflow:
1. The Lambda function is triggered (e.g., on a scheduled basis).
2. It connects to the Twitter API and retrieves tweets from the specified user.
3. It filters for recent tweets and analyzes their sentiment.
4. It uploads the processed tweets to S3 and saves them in the PostgreSQL database.
5. A log of the execution is generated, and success is reported if everything works.

### Dependencies:
- `twython`: For interacting with the Twitter API.
- `nltk`: For sentiment analysis using VADER.
- `pandas`: For handling data frames and inserting data into the database.
- `psycopg2`: For connecting to and interacting with the PostgreSQL database.
- `boto3`: For interacting with AWS services, specifically S3.

### Error Handling:
- The Lambda function catches errors during execution, logs them, and ensures the DB connection is properly closed to prevent locking issues.
