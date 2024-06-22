# Use an official Python runtime as a base image
FROM python:3.12-slim

# Set the working directory in the container
WORKDIR /app

# Copy the requirements files to the container's workspace
COPY requirements.txt requirements.txt

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Copy the local package files to the container's workspace
COPY . /app

# Make port 80 available to the world outside this container
EXPOSE 80

# Run main.py when the container launches
CMD ["python", "main.py"]
