FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Install dependencies first to leverage Docker cache
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose the required port
EXPOSE 5000

# Set the default command
CMD ["python", "app.py"]
