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


