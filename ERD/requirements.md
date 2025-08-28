erDiagram
    USER {
        CHAR_36 user_id PK "PRIMARY KEY, UUID"
        VARCHAR_50 first_name "NOT NULL"
        VARCHAR_50 last_name "NOT NULL"
        VARCHAR_255 email UK "UNIQUE, NOT NULL, INDEXED"
        VARCHAR_255 password_hash "NOT NULL"
        VARCHAR_20 phone_number "NULL"
        ENUM role "guest|host|admin, NOT NULL"
        TIMESTAMP created_at "DEFAULT CURRENT_TIMESTAMP"
    }

    PROPERTY {
        CHAR_36 property_id PK "PRIMARY KEY, UUID"
        CHAR_36 host_id FK "FOREIGN KEY -> User(user_id)"
        VARCHAR_200 name "NOT NULL"
        TEXT description "NOT NULL"
        VARCHAR_255 location "NOT NULL, INDEXED"
        DECIMAL_10_2 price_per_night "NOT NULL, CHECK > 0, INDEXED"
        TIMESTAMP created_at "DEFAULT CURRENT_TIMESTAMP, INDEXED"
        TIMESTAMP updated_at "ON UPDATE CURRENT_TIMESTAMP"
    }

    BOOKING {
        CHAR_36 booking_id PK "PRIMARY KEY, UUID"
        CHAR_36 property_id FK "FOREIGN KEY -> Property(property_id)"
        CHAR_36 user_id FK "FOREIGN KEY -> User(user_id)"
        DATE start_date "NOT NULL, CHECK >= CURDATE()"
        DATE end_date "NOT NULL, CHECK > start_date"
        DECIMAL_10_2 total_price "NOT NULL, CHECK > 0"
        ENUM status "pending|confirmed|canceled, NOT NULL, INDEXED"
        TIMESTAMP created_at "DEFAULT CURRENT_TIMESTAMP, INDEXED"
    }

    PAYMENT {
        CHAR_36 payment_id PK "PRIMARY KEY, UUID"
        CHAR_36 booking_id FK "FOREIGN KEY -> Booking(booking_id), UNIQUE"
        DECIMAL_10_2 amount "NOT NULL, CHECK > 0"
        TIMESTAMP payment_date "DEFAULT CURRENT_TIMESTAMP, INDEXED"
        ENUM payment_method "credit_card|paypal|stripe, NOT NULL, INDEXED"
    }

    REVIEW {
        CHAR_36 review_id PK "PRIMARY KEY, UUID"
        CHAR_36 property_id FK "FOREIGN KEY -> Property(property_id)"
        CHAR_36 user_id FK "FOREIGN KEY -> User(user_id)"
        INTEGER rating "NOT NULL, CHECK 1-5, INDEXED"
        TEXT comment "NOT NULL"
        TIMESTAMP created_at "DEFAULT CURRENT_TIMESTAMP, INDEXED"
    }

    MESSAGE {
        CHAR_36 message_id PK "PRIMARY KEY, UUID"
        CHAR_36 sender_id FK "FOREIGN KEY -> User(user_id)"
        CHAR_36 recipient_id FK "FOREIGN KEY -> User(user_id)"
        TEXT message_body "NOT NULL"
        TIMESTAMP sent_at "DEFAULT CURRENT_TIMESTAMP, INDEXED"
    }

    %% Primary Relationships
    USER ||--o{ PROPERTY : "hosts (1:M) - CASCADE DELETE"
    USER ||--o{ BOOKING : "makes (1:M) - CASCADE DELETE"
    USER ||--o{ REVIEW : "writes (1:M) - CASCADE DELETE"
    USER ||--o{ MESSAGE : "sends (1:M) - CASCADE DELETE"
    USER ||--o{ MESSAGE : "receives (1:M) - CASCADE DELETE"
    
    PROPERTY ||--o{ BOOKING : "booked_for (1:M) - CASCADE DELETE"
    PROPERTY ||--o{ REVIEW : "reviewed_for (1:M) - CASCADE DELETE"
    
    BOOKING ||--|| PAYMENT : "paid_by (1:1) - CASCADE DELETE"

    %% Business Rules and Constraints
    USER {
        string constraint_email "UNIQUE INDEX on email"
        string constraint_role "ENUM: guest, host, admin"
    }
    
    PROPERTY {
        string constraint_price "CHECK: price_per_night > 0"
        string index_host_price "COMPOSITE INDEX: host_id + price_per_night"
    }
    
    BOOKING {
        string constraint_dates "CHECK: end_date > start_date"
        string constraint_future "CHECK: start_date >= CURDATE()"
        string constraint_price "CHECK: total_price > 0"
        string unique_property_dates "UNIQUE: property_id + start_date + end_date"
        string index_dates "COMPOSITE INDEX: start_date + end_date"
        string index_user_status "COMPOSITE INDEX: user_id + status"
    }
    
    PAYMENT {
        string constraint_amount "CHECK: amount > 0"
        string constraint_unique_booking "UNIQUE: booking_id (1:1 relationship)"
    }
    
    REVIEW {
        string constraint_rating "CHECK: rating BETWEEN 1 AND 5"
        string unique_user_property "UNIQUE: user_id + property_id"
    }
    
    MESSAGE {
        string constraint_different_users "CHECK: sender_id != recipient_id"
        string index_conversation "COMPOSITE INDEX: sender_id + recipient_id + sent_at"
    }
