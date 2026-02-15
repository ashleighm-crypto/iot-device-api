# IoT Device Management REST API

A Flask-based REST API for managing IoT devices with full CRUD operations, input validation, and persistent storage.

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![Flask](https://img.shields.io/badge/flask-2.0+-green.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 📋 Project Overview

This is my **first production-style REST API**. Built as a learning project to understand:
- RESTful API design principles
- Data Access Layer (DAL) architecture pattern
- Input validation and error handling
- Persistent data storage

**What it does:** Manages smart home devices (lights, sensors, humidifiers) through HTTP endpoints.

## 🎯 Key Features

✅ **Full CRUD Operations** - Create, Read, Update, Delete devices  
✅ **Data Validation** - Marshmallow schemas validate all inputs  
✅ **Error Handling** - Proper HTTP status codes and error messages  
✅ **Duplicate Prevention** - Checks for existing IDs before creating  
✅ **Partial Updates** - Update only specific fields  
✅ **Persistent Storage** - Data survives server restarts  
✅ **Separation of Concerns** - Clean DAL architecture  

## 🏗️ Architecture
```
┌─────────────────────────────────────┐
│         api.py (API Layer)          │
│  • Routes & HTTP logic              │
│  • Request/response handling        │
│  • Input validation                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│         dal.py (Data Layer)         │
│  • Database operations              │
│  • Business logic                   │
│  • Data persistence                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      Shelve Database (Storage)      │
└─────────────────────────────────────┘
```

**Why this matters:** Each layer has one responsibility. Change the database? Only touch `dal.py`. Change API routes? Only touch `api.py`.

## 🛠️ Tech Stack

- **Python 3.8+** - Core programming language
- **Flask** - Lightweight WSGI web framework
- **Marshmallow** - Object serialization and validation
- **Shelve** - Python's built-in persistent dictionary

## 📡 API Endpoints

### Device Collection (`/items`)

| Method | Endpoint | Description | Status Codes |
|--------|----------|-------------|--------------|
| `GET` | `/items` | Retrieve all devices | 200 |
| `POST` | `/items` | Create a new device | 201, 400 |

### Single Device (`/items/<id>`)

| Method | Endpoint | Description | Status Codes |
|--------|----------|-------------|--------------|
| `GET` | `/items/<id>` | Get device by ID | 200, 404 |
| `PUT` | `/items/<id>` | Update device | 200, 400, 404 |
| `DELETE` | `/items/<id>` | Delete device | 200, 404 |

## 🚀 Quick Start

### Prerequisites
```bash
Python 3.8 or higher
pip (Python package manager)
```

### Installation
```bash
# Clone the repository
git clone https://github.com/yourusername/iot-device-api.git
cd iot-device-api

# Install dependencies
pip install flask marshmallow

# Run the server
python api.py
```

Server runs at `http://localhost:5000`

## 💻 Usage Examples

### Create a Device
```bash
curl -X POST http://localhost:5000/items \
  -H "Content-Type: application/json" \
  -d '{
    "id": "004",
    "name": "Smart Lock",
    "location": "front door",
    "status": "locked"
  }'
```

**Response (201):**
```json
{
  "Posted a device": {
    "id": "004",
    "name": "Smart Lock",
    "location": "front door",
    "status": "locked"
  }
}
```

### Get All Devices
```bash
curl http://localhost:5000/items
```

**Response (200):**
```json
{
  "items": {
    "001": {"id": "001", "name": "Light bulb", "location": "hall", "status": "off"},
    "002": {"id": "002", "name": "Humidity_sensor", "location": "bedroom", "status": "on"}
  }
}
```

### Get Single Device
```bash
curl http://localhost:5000/items/001
```

**Response (200):**
```json
{
  "id": "001",
  "name": "Light bulb",
  "location": "hall",
  "status": "off"
}
```

### Update a Device
```bash
curl -X PUT http://localhost:5000/items/001 \
  -H "Content-Type: application/json" \
  -d '{"status": "on"}'
```

**Response (200):**
```json
{
  "updated device": "001"
}
```

### Delete a Device
```bash
curl -X DELETE http://localhost:5000/items/001
```

**Response (200):**
```json
{
  "message": "001 deleted"
}
```

## 🎓 What I Learned

### 1. **Separation of Concerns**
The DAL pattern taught me that **mixing database code with API code is a mistake**. By separating them:
- Testing becomes easier
- Code is more maintainable
- Swapping databases doesn't break the API

### 2. **Error Handling is UX**
Good error messages help developers using your API:
- `"Device with this ID already exists"` is better than `"Bad request"`
- Proper status codes (404, 400, 201) communicate what happened

### 3. **Validation is Non-Negotiable**
Users will send bad data. Always validate:
- Required fields present?
- Data types correct?
- IDs don't already exist?

### 4. **Context Managers Prevent Bugs**
Using `with` statements ensures database connections close properly:
```python
with pull_db() as shelf:
    # Do database operations
# Guaranteed to close, even if errors occur
```

### 5. **REST Conventions Matter**
- `POST` creates → return 201
- `GET` retrieves → return 200
- `PUT` updates → return 200
- `DELETE` removes → return 200
- Resource not found? → return 404
- Bad input? → return 400

## 📂 Project Structure
```
iot-device-api/
│
├── api.py              # Flask routes and HTTP handling
├── dal.py              # Database access layer
├── storage.db          # Shelve database file (auto-generated)
├── README.md           # This file
└── requirements.txt    # Python dependencies
```

## 🔍 Code Highlights

### Input Validation (Marshmallow)
```python
class DeviceSchema(Schema):
    id = fields.Str(required=True)
    name = fields.Str(required=True)
    location = fields.Str(required=True)
    status = fields.Str(required=True)
```

### Duplicate Prevention
```python
def post(args):
    with pull_db() as shelf:
        if args['id'] in shelf:
            return {'error': 'Device with this ID already exists.'}
        shelf[args['id']] = args
```

### Clean Error Handling
```python
if result is None:
    return jsonify({'message': 'Device not found'}), 404
return jsonify(result), 200
```

## 🐛 Error Handling

The API returns descriptive error messages:

| Scenario | Status Code | Response |
|----------|-------------|----------|
| Device not found | 404 | `{"message": "Device not found"}` |
| Duplicate ID | 400 | `{"message": "Device with this ID already exists"}` |
| Invalid input | 400 | `{"field": ["Error message"]}` |
| Success | 200/201 | Device data or confirmation |

## 🚧 Limitations & Future Improvements

**Current Limitations:**
- No authentication/authorization
- Shelve database (not production-grade)
- No pagination for large datasets
- No rate limiting

**Planned Enhancements:**
- [ ] Add JWT authentication
- [ ] Migrate to PostgreSQL
- [ ] Implement pagination
- [ ] Add rate limiting
- [ ] Write unit tests
- [ ] Add API documentation (Swagger)
- [ ] Containerize with Docker

## 🤝 Contributing

This is a learning project, but feedback is welcome! If you spot issues or have suggestions:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## 📝 License

MIT License - feel free to use this code for learning purposes.

## 👨‍💻 About This Project

This is my **first REST API** built from scratch. I created it to understand:
- How professional APIs are structured
- Why architecture patterns matter
- How to write maintainable backend code

**Connect with me:**
- LinkedIn: https://www.linkedin.com/in/ashleigh-mandonga/
- Email: ashleighmandonga@gmail.com
---

⭐ If this helped you learn, consider starring the repo!
