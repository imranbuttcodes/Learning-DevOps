FROM node:20-alpine

# Set your environment variables
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=qwerty

# 1. Set the working directory (this creates the folder automatically!)
WORKDIR /testapp

# 2. Copy package files first
COPY package*.json ./

# 3. CRITICAL STEP: Download and install express/mongodb inside the image
RUN npm install

# 4. Copy the rest of your local server code files
COPY . .

# 5. Boot up the server cleanly
CMD ["node", "server.js"]
