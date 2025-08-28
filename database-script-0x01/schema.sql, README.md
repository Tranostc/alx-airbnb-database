--Users
CREATE TABLE users (
  user_id         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  role_id         SMALLINT UNSIGNED NOT NULL,
  full_name       VARCHAR(120) NOT NULL,
  email           VARCHAR(255) NOT NULL,
  phone           VARCHAR(32) NULL,
  password_hash   CHAR(60) NOT NULL, -- bcrypt/argon2 hash length
  created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (user_id),
  UNIQUE KEY uq_users_email (email),
  KEY idx_users_role (role_id),
  CONSTRAINT fk_users_role
    FOREIGN KEY (role_id) REFERENCES roles(role_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Locations (lookup; normalized)
CREATE TABLE locations (
  location_id  INT UNSIGNED NOT NULL AUTO_INCREMENT,
  city         VARCHAR(100) NOT NULL,
  state        VARCHAR(100) NULL,
  country      VARCHAR(100) NOT NULL,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (location_id),
  UNIQUE KEY uq_locations (city, state, country)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Properties
CREATE TABLE properties (
  property_id       BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  host_user_id      BIGINT UNSIGNED NOT NULL,
  title             VARCHAR(150) NOT NULL,
  description       TEXT NULL,
  location_id       INT UNSIGNED NOT NULL,
  price_per_night   DECIMAL(10,2) NOT NULL,
  status            ENUM('available','unavailable') NOT NULL DEFAULT 'available',
  created_at        TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at        TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (property_id),
  KEY idx_properties_host (host_user_id),
  KEY idx_properties_location (location_id),
  KEY idx_properties_price (price_per_night),
  KEY idx_properties_status (status),
  CONSTRAINT chk_properties_price CHECK (price_per_night >= 0),
  CONSTRAINT fk_properties_host
    FOREIGN KEY (host_user_id) REFERENCES users(user_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
  CONSTRAINT fk_properties_location
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Payment Methods (lookup)
CREATE TABLE payment_methods (
  payment_method_id  SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
  method_name        VARCHAR(40) NOT NULL,
  created_at         TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (payment_method_id),
  UNIQUE KEY uq_payment_methods_name (method_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Bookings
CREATE TABLE bookings (
  booking_id     BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  guest_user_id  BIGINT UNSIGNED NOT NULL,
  property_id    BIGINT UNSIGNED NOT NULL,
  check_in_date  DATE NOT NULL,
  check_out_date DATE NOT NULL,
  total_price    DECIMAL(10,2) NOT NULL,
  booking_status ENUM('pending','confirmed','cancelled') NOT NULL DEFAULT 'pending',
  created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (booking_id),
  KEY idx_bookings_guest (guest_user_id),
  KEY idx_bookings_property (property_id),
  KEY idx_bookings_dates (property_id, check_in_date, check_out_date),
  CONSTRAINT chk_bookings_dates CHECK (check_in_date < check_out_date),
  CONSTRAINT chk_bookings_price CHECK (total_price >= 0),
  CONSTRAINT fk_bookings_guest
    FOREIGN KEY (guest_user_id) REFERENCES users(user_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
  CONSTRAINT fk_bookings_property
    FOREIGN KEY (property_id) REFERENCES properties(property_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Payments (allowing partial/multiple payments)
CREATE TABLE payments (
  payment_id         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  booking_id         BIGINT UNSIGNED NOT NULL,
  payment_method_id  SMALLINT UNSIGNED NOT NULL,
  payment_date       DATETIME NOT NULL,
  amount             DECIMAL(10,2) NOT NULL,
  payment_status     ENUM('successful','failed','refunded') NOT NULL,
  transaction_ref    VARCHAR(100) NULL,
  created_at         TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (payment_id),
  UNIQUE KEY uq_payments_txn_ref (transaction_ref),
  KEY idx_payments_booking (booking_id),
  KEY idx_payments_date (payment_date),
  CONSTRAINT chk_payments_amount CHECK (amount > 0),
  CONSTRAINT fk_payments_booking
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
  CONSTRAINT fk_payments_method
    FOREIGN KEY (payment_method_id) REFERENCES payment_methods(payment_method_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Reviews (1:1 with bookings -> one review per booking)
CREATE TABLE reviews (
  review_id     BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  booking_id    BIGINT UNSIGNED NOT NULL,
  rating        TINYINT UNSIGNED NOT NULL,
  comment       TEXT NULL,
  review_date   DATE NOT NULL,
  created_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (review_id),
  UNIQUE KEY uq_reviews_booking (booking_id),
  CONSTRAINT chk_reviews_rating CHECK (rating BETWEEN 1 AND 5),
  CONSTRAINT fk_reviews_booking
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id)
    ON UPDATE CASCADE
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Suggested additional indexes for common query patterns:
--  * Searching properties by location + price range + availability status
--  * Finding bookings for a property by date window (idx_bookings_dates)
--  * Looking up a user's bookings (idx_bookings_guest)
--  * Payment reporting by date (idx_payments_date)

-- Seed minimal lookup data (optional)
INSERT INTO roles (role_name) VALUES ('guest'), ('host'), ('admin');
INSERT INTO payment_methods (method_name) VALUES ('card'), ('paypal'), ('eft');
"""

--Readme.md

## Objective
To normalize the AirBnB database schema to ensure it is in the Third Normal Form (3NF) to reduce redundancy and improve data integrity.

## Normalization Steps

### First Normal Form (1NF)
- **Criteria**: All attributes must contain atomic values, each column must have unique names, and the order of data storage does not matter.
- **Review**: The schema is in 1NF as all attributes are atomic and contain single values.

### Second Normal Form (2NF)
- **Criteria**: The schema must be in 1NF, and all non-key attributes must be fully functionally dependent on the primary key.
- **Review**: The schema is in 2NF as all non-key attributes depend on their respective primary keys.

### Third Normal Form (3NF)
- **Criteria**: The schema must be in 2NF, and there should be no transitive dependencies.
- **Review**: The schema is in 3NF as all attributes depend solely on their primary keys, with no transitive dependencies.

## Conclusion
The AirBnB database schema is already in Third Normal Form (3NF). No additional changes were necessary, as all tables are structured to minimize redundancy and maintain data integrity.
