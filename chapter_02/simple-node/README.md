### Run the following command to create simple-node Docker image
```bash
docker build -t simple-node .
```

### Run image
```bash
docker run --rm -p 3000:3000 simple-node
```