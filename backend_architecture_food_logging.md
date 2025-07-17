# Self-Recording Health Mobile App – Backend Design (Food Logging Only)

## 1. Database Model (SQL)

### Users Table (`users`)
| Column Name   | Data Type     | Description                |
|---------------|--------------|----------------------------|
| id            | INTEGER      | User ID (Primary Key)      |
| phone_number  | VARCHAR(20)  | Phone number               |
| created_at    | TIMESTAMP    | Registration date          |

### Meals Table (`meals`)
| Column Name   | Data Type     | Description                        |
|---------------|--------------|------------------------------------|
| id            | INTEGER      | Meal ID (Primary Key)              |
| user_id       | INTEGER      | User ID (Foreign Key)              |
| meal_time     | TIMESTAMP    | Date and time of the meal          |
| meal_type     | VARCHAR(20)  | Type (breakfast/lunch/dinner/snack)|
| notes         | TEXT         | Notes                              |
| created_at    | TIMESTAMP    | Record creation date               |

### Food Catalog Table (`food_catalog`)
| Column Name   | Data Type     | Description                |
|---------------|--------------|----------------------------|
| id            | INTEGER      | Food ID (Primary Key)      |
| name          | VARCHAR(100) | Name of the food           |
| calories      | FLOAT        | Calories per 100g/unit     |
| protein       | FLOAT        | Protein (g) per 100g/unit  |
| carbs         | FLOAT        | Carbs (g) per 100g/unit    |
| fat           | FLOAT        | Fat (g) per 100g/unit      |
| fiber         | FLOAT        | Fiber (g) per 100g/unit    |
| sugar         | FLOAT        | Sugar (g) per 100g/unit    |
| sodium        | FLOAT        | Sodium (mg) per 100g/unit  |
| ...           | ...          | Additional nutrients       |

### Meal Items Table (`meal_items`)
| Column Name   | Data Type     | Description                        |
|---------------|--------------|------------------------------------|
| id            | INTEGER      | Item ID (Primary Key)              |
| meal_id       | INTEGER      | Meal ID (Foreign Key)              |
| food_id       | INTEGER      | Food ID (Foreign Key to food_catalog) |
| quantity      | FLOAT        | Quantity                           |
| unit          | VARCHAR(20)  | Unit (g, ml, piece, etc.)          |
| calories      | FLOAT        | Calories (optional, override)      |
| protein       | FLOAT        | Protein (g, optional, override)    |
| carbs         | FLOAT        | Carbs (g, optional, override)      |
| fat           | FLOAT        | Fat (g, optional, override)        |
| ...           | ...          | Additional nutrients (optional)    |

---

## 2. API List (OpenAPI Style)

> **Note:** Starting from this implementation, each meal item (meal_item) references the food catalog using food_id. You may send custom nutritional values (override), but by default, values are calculated automatically based on the catalog.

### 2.1 Login
- **POST /api/auth/login**
  - **Request Body:**  
    ```json
    { "phone_number": "string" }
    ```
  - **Response:**  
    ```json
    { "otp_sent": true }
    ```

### 2.2 OTP Verification
- **POST /api/auth/verify**
  - **Request Body:**  
    ```json
    { "phone_number": "string", "otp": "string" }
    ```
  - **Response:**  
    ```json
    { "token": "string" }
    ```

### 2.3 Create New Meal
- **POST /api/meals**
  - **Header:** Authorization: Bearer {token}
  - **Request Body:**
    ```json
    {
      "meal_time": "datetime",
      "meal_type": "string",
      "notes": "string",
      "items": [
        {
          "food_id": 12,
          "quantity": 150,
          "unit": "g",
          "calories": 200,   // Optional, auto-calculated if not provided
          "protein": 15,     // Optional, auto-calculated if not provided
          "carbs": 30,       // Optional, auto-calculated if not provided
          "fat": 5           // Optional, auto-calculated if not provided
        }
      ]
    }
    ```
  - **Response:**
    ```json
    { "meal_id": 101 }
    ```
  - **Note:** If nutritional values are not provided, the system will calculate them automatically based on the food catalog values and the specified quantity/unit.

### 2.4 Get Meals List
- **GET /api/meals**
  - **Header:** Authorization: Bearer {token}
  - **Query Parameters:**  
    - from_date (optional)
    - to_date (optional)
  - **Response:**
    ```json
    [
      {
        "id": 101,
        "meal_time": "2024-06-01T12:00:00Z",
        "meal_type": "lunch",
        "notes": "Post-workout meal",
        "items": [
          {
            "food_id": 12,
            "food_name": "Chicken Breast",
            "quantity": 150,
            "unit": "g",
            "calories": 200,
            "protein": 15,
            "carbs": 30,
            "fat": 5
          }
        ]
      }
    ]
    ```
  - **Note:** food_name and nutritional values are fetched automatically from the catalog based on food_id, unless an override was provided.

### 2.5 Update Meal
- **PUT /api/meals/{meal_id}**
  - **Header:** Authorization: Bearer {token}
  - **Request Body:** Same as POST /api/meals
  - **Response:**
    ```json
    { "success": true }
    ```

### 2.6 Delete Meal
- **DELETE /api/meals/{meal_id}**
  - **Header:** Authorization: Bearer {token}
  - **Response:**
    ```json
    { "success": true }
    ```

### 2.7 Food Catalog Endpoints
- **GET /api/foods**
  - **Description:** Retrieve all foods from the catalog (for use in the app, e.g., for search/selection).
  - **Response:**
    ```json
    [
      {
        "id": 12,
        "name": "Chicken Breast",
        "calories": 130,
        "protein": 22,
        "carbs": 0,
        "fat": 3,
        "fiber": 0,
        "sugar": 0,
        "sodium": 50
      }
    ]
    ```
- **POST /api/foods** (admin only)
  - **Description:** Add a new food to the catalog
  - **Request Body:**
    ```json
    {
      "name": "Chicken Breast",
      "calories": 130,
      "protein": 22,
      "carbs": 0,
      "fat": 3,
      "fiber": 0,
      "sugar": 0,
      "sodium": 50
    }
    ```
  - **Response:**
    ```json
    { "food_id": 12 }
    ```

---

## 2.x Example: Automatic Nutritional Value Calculation
> For example, if you select food_id=12 (Chicken Breast, 130 calories per 100g) and send quantity=150, the system will return:
> calories = 130 * 1.5 = 195
> protein = 22 * 1.5 = 33
> and so on.

---

## 3. Architecture Diagram

```