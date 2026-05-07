FROM python:3.11

# create a working directory
WORKDIR /app/

# Copy only the file which wont be impacted due to the change
COPY ./flask_app/requirements.txt .
# run commands
RUN pip install --no-cache-dir -r requirements.txt
RUN python -m nltk.downloader stopwords wordnet

# Copy models
COPY ./flask_app/ .
COPY ./flask_app/templates/ ./templates/
COPY ./models/ ./models/

# add default value of env variable
ENV PORT=5000

# EXPOSE ports
EXPOSE 5000

# CMD
CMD ["python","app.py"]