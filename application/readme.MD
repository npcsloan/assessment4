# **Twoge**

<h1 align="center">
	<img
		width="500"
		alt="Project Twoge"
		src="./static/img/twoge-cover.png">
</h1>

Twoge is a social media platform dedicated solely to tweets about Dodge. This application is built using Flask, SQLAlchemy, and PostgreSQL.

<h1 align="center">
	<img
		width="100"
		alt="Project Twoge"
		src="./static/img/twoge.png">
</h1>

## **Prerequisites**

Before you can launch Twoge, you will need to have the following software installed on your system:

- Python 3
- Flask
- SQLAlchemy
- PostgreSQL

## **Getting started**

To get started with Twoge, follow these steps:

1. Clone the repository to your local machine.
2. Create a virtual environment for the project and activate it.
3. Install the required packages using **`pip install -r requirements.txt`**.
4. Create a PostgreSQL database for the application.
5. Set the **`DATABASE_URL`** environment variable to the URL of your PostgreSQL database.
6. Run the application using **`python app.py`**.
7. Navigate to **`http://localhost:5000`** in your web browser to use Twoge.

## **Table attributes**

Twoge has a single table in its database, which stores information about each tweet. The table has the following attributes:

- **`date_posted`**: The date and time when the tweet was posted.
- **`author`**: The user who posted the tweet.
- **`title`**: The title of the tweet.
- **`content`**: The content of the tweet.