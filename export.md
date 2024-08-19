To manage configuration files based on environment values in Python 3.12, you can follow these steps:

1. **Create Configuration Files**: Define separate configuration files for different environments (e.g., `config_dev.json`, `config_prod.json`, etc.).

2. **Load Environment Variables**: Use environment variables to determine which configuration file to load.

3. **Load Configuration**: Load the appropriate configuration file based on the environment variable.

4. **Use the Configuration in Your Code**: Access the configuration settings in your main application code.

### Example Implementation

#### 1. Create Configuration Files

Create separate configuration files for each environment:

**config_dev.json**:
```json
{
    "database": {
        "host": "localhost",
        "port": 5432,
        "username": "dev_user",
        "password": "dev_pass"
    },
    "debug": true
}
```

**config_prod.json**:
```json
{
    "database": {
        "host": "prod.db.server",
        "port": 5432,
        "username": "prod_user",
        "password": "prod_pass"
    },
    "debug": false
}
```

#### 2. Load Environment Variable

You can use the `os` module to load the environment variable that specifies the environment:

```python
import os

env = os.getenv('APP_ENV', 'dev')  # Default to 'dev' if APP_ENV is not set
```

#### 3. Load Configuration Based on Environment

Use the `json` module to load the configuration file:

```python
import json

def load_config(env):
    config_file = f"config_{env}.json"
    try:
        with open(config_file, 'r') as file:
            config = json.load(file)
            return config
    except FileNotFoundError:
        raise Exception(f"Configuration file '{config_file}' not found.")
    except json.JSONDecodeError:
        raise Exception(f"Error decoding JSON from '{config_file}'.")

config = load_config(env)
```

#### 4. Use Configuration in Your Main Code

You can now use the loaded configuration in your main application code:

```python
def main():
    db_config = config['database']
    
    print(f"Connecting to database at {db_config['host']}:{db_config['port']}")
    # Use db_config['username'], db_config['password'], etc.
    
    if config['debug']:
        print("Debug mode is enabled")
    
    # Continue with the rest of your application logic

if __name__ == "__main__":
    main()
```

### Running the Application

Set the environment variable `APP_ENV` to switch between configurations:

```bash
export APP_ENV=prod
python main.py
```

Or run it with the environment variable inline:

```bash
APP_ENV=prod python main.py
```

### Summary

- **Environment Variable**: Use environment variables to select the appropriate configuration.
- **Configuration Files**: Store configurations in separate files and load them dynamically.
- **Main Application**: Use the loaded configuration in your main application logic.

This approach allows for flexible and environment-specific configuration management in your Python applications.
