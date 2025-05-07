# OS_Final AWS project Movie Web Application using EC2, PostgreSQL (RDS), Flask Python
- Amazon EC2 (Ubuntu Server)
- Amazon RDS (PostgreSQL)
- Python + Flask Framework
- HTML + Bootstrap
- psycopg2
- Git + GitHub
  
# Application Functionality

- Connects to an RDS PostgreSQL database (`movie`)
- Displays movie data in a styled HTML page
- Backend developed in Python using Flask
- EC2 used as the hosting server
- Optionally serve static content via Amazon S3

# Steps to Run
### 1. SSH into my EC2 instance
ssh -i "gozal_key.pem" ubuntu@43.201.86.41

### 2. Create and activate a virtual environment
python3 -m venv myenv
source myenv/bin/activate

### 3. Run the app
python3 app.py

### 4. Database setup
i have created postgres database from terminal
postgres=> \d movies
                                             Table "public.movies"
       Column        |          Type          | Collation | Nullable |                 Default
---------------------+------------------------+-----------+----------+------------------------------------------
 movie_id            | integer                |           | not null | nextval('movies_movie_id_seq'::regclass)
 movie_title         | character varying(255) |           | not null |
 release_year        | integer                |           | not null |
 genre               | character varying(255) |           |          |
 director            | character varying(255) |           |          |
 box_office_earnings | bigint                 |           |          |
Indexes:
    "movies_pkey" PRIMARY KEY, btree (movie_id)
### 5. seting up EC2 Python Backend
Launched an EC2 instance Ubuntu
Installed Python and Flask w/ the codes below:
sudo yum update -y
sudo yum install python3 -y
pip3 install flask pymysql flask-cors
### 6. Created app.py
from flask import Flask, jsonify, request
import psycopg2
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# Database configuration
DB_HOST = 'db-gozal.clyucs4e44b4.ap-northeast-2.rds.amazonaws.com'  # Replace with your RDS endpoint
DB_NAME = 'postgres'
DB_USER = 'postgres'    # Replace with your RDS username
DB_PASS = 'postgres'  # Replace with your RDS password
DB_PORT = '5432'

def get_db_connection():
    """
    Establishes a connection to the PostgreSQL database.
    Handles connection errors and returns a connection object.
    """
    try:
        conn = psycopg2.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASS,
            port=DB_PORT
        )
        return conn
    except psycopg2.Error as e:
        print(f"Error connecting to the database: {e}")
        return None

@app.route('/movies', methods=['GET'])
def get_movies():
    """
    Retrieves all movies from the 'movies' table.
    Returns a JSON list of movies.
    """
    conn = get_db_connection()
    if conn is None:
        return jsonify({'message': 'Failed to connect to the database'}), 500
    cur = conn.cursor()
    try:
        cur.execute("SELECT movie_id, movie_title, release_year, genre, director, box_office_earnings FROM movies")
        movies = cur.fetchall()
        movies_list = [{'movie_id': movie[0], 'movie_title': movie[1], 'release_year': movie[2],
                        'genre': movie[3], 'director': movie[4], 'box_office_earnings': movie[5]} for movie in movies]
        return jsonify(movies_list)
    except psycopg2.Error as e:
        print(f"Error fetching movies: {e}")
        return jsonify({'message': f'Error fetching movies: {e}'}), 500
    finally:
        cur.close()
        conn.close()

@app.route('/movies', methods=['POST'])
def add_movie():
    """
    Adds a new movie to the 'movies' table.
    Expects movie data in JSON format in the request body.
    Returns a JSON message indicating success or failure.
    """
    data = request.get_json()
    movie_title = data.get('movie_title')
    release_year = data.get('release_year')
    genre = data.get('genre')
    director = data.get('director')
    box_office_earnings = data.get('box_office_earnings')

    if not all([movie_title, release_year, genre, director, box_office_earnings]):
        return jsonify({'message': 'Missing required fields'}), 400

    conn = get_db_connection()
    if conn is None:
        return jsonify({'message': 'Failed to connect to the database'}), 500
    cur = conn.cursor()
    try:
        cur.execute(
            "INSERT INTO movies (movie_title, release_year, genre, director, box_office_earnings) VALUES (%s, %s, %s, %s, %s)",
            (movie_title, release_year, genre, director, box_office_earnings)
        )
        conn.commit()
        return jsonify({'message': 'Movie added successfully'}), 201
    except psycopg2.Error as e:
        conn.rollback()
        print(f"Error adding movie: {e}")
        return jsonify({'message': f'Error adding movie: {e}'}), 500
    finally:
        cur.close()
        conn.close()

@app.route('/movies/<int:movie_id>', methods=['DELETE'])
def delete_movie(movie_id):
    """
    Deletes a movie from the 'movies' table by movie ID.
    Returns a JSON message indicating success or failure.
    """
    conn = get_db_connection()
    if conn is None:
        return jsonify({'message': 'Failed to connect to the database'}), 500
    cur = conn.cursor()
    try:
        cur.execute("DELETE FROM movies WHERE movie_id = %s", (movie_id,))
        if cur.rowcount > 0:
            conn.commit()
            return jsonify({'message': f'Movie with ID {movie_id} deleted successfully'}), 200
        else:
            return jsonify({'message': f'Movie with ID {movie_id} not found'}), 404
    except psycopg2.Error as e:
        conn.rollback()
        print(f"Error deleting movie: {e}")
        return jsonify({'message': f'Error deleting movie: {e}'}), 500
    finally:
        cur.close()
        conn.close()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)


### 7. runned app: sudo python3 app.py
### 8. have Createe Frontend Website in S3


