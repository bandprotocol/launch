# Import Bothan private key

If you are setting up a new node or have lost Bothan data, you can import the Bothan keys to resume participation in the Bothan monitoring.

### Step 1: Start Bothan

Start Bothan if not already.

```bash
sudo docker pull bandprotocol/bothan-api:v0.0.1-beta.3
CONTAINER_ID=$(sudo docker run --restart always --log-opt max-size=50m --log-opt max-file=5 -d --name bothan -v "$HOME/.bothan:/root/.bothan" -p 50051:50051 bandprotocol/bothan-api:v0.0.1-beta.3)
```

### Step 2: Import private key

```bash
sudo docker exec -it bothan /bin/sh -c "bothan key import --override"
```

You will be prompted to enter the private key. Please provide the private key you have previously saved.

### Step 3: Restart bothan

```bash
sudo docker restart bothan
```

You should then be able to continue sending Bothan monitoring data using the same account.
