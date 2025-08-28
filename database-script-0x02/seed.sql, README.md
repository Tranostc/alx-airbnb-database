-- Sample Users
INSERT INTO User (user_id, first_name, last_name, email, password_hash, phone_number, role)
VALUES
    (uuid_generate_v4(), 'Alice', 'Johnson', 'alice.johnson@example.com', 'hashed_password_1', '1234567890', 'guest'),
    (uuid_generate_v4(), 'Bob', 'Smith', 'bob.smith@example.com', 'hashed_password_2', '0987654321', 'host'),
    (uuid_generate_v4(), 'Charlie', 'Brown', 'charlie.brown@example.com', 'hashed_password_3', '5678901234', 'guest'),
    (uuid_generate_v4(), 'Diana', 'Prince', 'diana.prince@example.com', 'hashed_password_4', '4321098765', 'admin'),
    (uuid_generate_v4(), 'Eve', 'Davis', 'eve.davis@example.com', 'hashed_password_5', '3216549870', 'host');

-- Sample Properties
INSERT INTO Property (property_id, host_id, name, description, location, pricepernight)
VALUES
    (uuid_generate_v4(), (SELECT user_id FROM User WHERE email = 'bob.smith@example.com'), 'Cozy Cottage', 'A cozy cottage in the woods.', '123 Forest Lane', 100.00),
    (uuid_generate_v4(), (SELECT user_id FROM User WHERE email = 'eve.davis@example.com'), 'Downtown Apartment', 'Modern apartment in the city center.', '456 City Road', 150.00),
    (uuid_generate_v4(), (SELECT user_id FROM User WHERE email = 'bob.smith@example.com'), 'Beach House', 'Beautiful beach house with ocean view.', '789 Ocean Drive', 200.00);

-- Sample Bookings
INSERT INTO Booking (booking_id, property_id, user_id, start_date, end_date, total_price, status)
VALUES
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Cozy Cottage'), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), '2025-09-01', '2025-09-05', 400.00, 'confirmed'),
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Downtown Apartment'), (SELECT user_id FROM User WHERE email = 'charlie.brown@example.com'), '2025-09-10', '2025-09-15', 750.00, 'pending'),
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Beach House'), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), '2025-09-20', '2025-09-25', 1000.00, 'confirmed');

-- Sample Payments
INSERT INTO Payment (payment_id, booking_id, amount, payment_method)
VALUES
    (uuid_generate_v4(), (SELECT booking_id FROM Booking WHERE status = 'confirmed' AND user_id = (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com') LIMIT 1), 400.00, 'credit_card'),
    (uuid_generate_v4(), (SELECT booking_id FROM Booking WHERE status = 'pending' AND user_id = (SELECT user_id FROM User WHERE email = 'charlie.brown@example.com') LIMIT 1), 750.00, 'paypal');

-- Sample Reviews
INSERT INTO Review (review_id, property_id, user_id, rating, comment)
VALUES
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Cozy Cottage'), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), 5, 'Absolutely loved this cozy cottage! Highly recommend.'),
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Downtown Apartment'), (SELECT user_id FROM User WHERE email = 'charlie.brown@example.com'), 4, 'Great location, but a bit noisy at night.'),
    (uuid_generate_v4(), (SELECT property_id FROM Property WHERE name = 'Beach House'), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), 5, 'Stunning views and very relaxing!');

-- Sample Messages
INSERT INTO Message (message_id, sender_id, recipient_id, message_body)
VALUES
    (uuid_generate_v4(), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), (SELECT user_id FROM User WHERE email = 'bob.smith@example.com'), 'Hi Bob, I loved your Cozy Cottage!'),
    (uuid_generate_v4(), (SELECT user_id FROM User WHERE email = 'bob.smith@example.com'), (SELECT user_id FROM User WHERE email = 'alice.johnson@example.com'), 'Thanks Alice! I hope to host you again soon.');
