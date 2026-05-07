FROM python:3.11

# create a working directory
WORKDIR /app/

# Copy app
COPY ./flask_app/ .

# Copy models
COPY ./models/ ./models/

# add default value of env variable
ENV PORT=5000

# run commands
RUN pip install --no-cache-dir -r requirements.txt
RUN python -m nltk.downloader stopwords wordnet

# EXPOSE ports
EXPOSE 5000

# CMD
CMD ["python","app.py"]