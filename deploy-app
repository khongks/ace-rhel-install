# Deploy App

1. Package application. Go to the project folder. Run the command.
  ```
  ibmint package --input-path . --output-bar-file FlightDetails.bar
  ```

1. Assume deploy on an integration server ISERVER01, managed by integration node INODE01
  ```
  ibmint deploy \
    --input-bar-file FlightDetails.bar \
    --output-host <node-host-name> \
    --output-port 4414 \
    --output-server ISERVER01 \
    --https \
    --insecure
  ```
