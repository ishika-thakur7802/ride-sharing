# ride-sharing

 1. User Registration and Login
   -   Rider and Driver  : Both need to register and log in using the platform’s mobile app or web interface.
   -   Backend  :
     - When a user registers, their information (e.g., name, email, phone, type of user) is stored in PostgreSQL.
     - The user’s login creates a JSON Web Token (JWT) to authenticate them during their session, enabling secure access to features.

2.   Requesting a Ride (Rider’s Perspective)  
   -   Rider  : 
     - Opens the app, selects their pick-up and drop-off locations, and clicks "Request Ride."
     - The app displays a waiting screen as it searches for available drivers.
   
   -   Backend Workflow  :
     1.   Ride Request  : This request is sent via a REST API to the backend.
     2.   Kafka Event  : The request triggers a Kafka event, which queues the ride request for processing.
     3.   Redis Matching  : The `Ride Matching Service` checks Redis to find available drivers within the rider’s vicinity.
     4.   Driver Match  : Once a match is found, the driver's details are returned to the rider app, and a WebSocket connection is established for real-time updates.

  # 3.   Receiving a Ride Request (Driver’s Perspective)  
   -   Driver  :
     - A driver logged into the app will receive a ride request notification.
     - The driver can either accept or reject the request. If accepted, the app displays the rider’s location and pickup details.
   
   -   Backend Workflow  :
     -   Kafka Notification  : When a match is made, an event is sent through Kafka to notify the driver.
     -   WebSocket Connection  : Upon acceptance, a WebSocket connection is established between the driver and rider for real-time updates (e.g., ETA, location).
     -   Redis Cache Update  : The driver’s availability status in Redis is updated to "engaged" until the ride is completed.

  # 4.   Tracking Location (Real-Time Updates)  
   -   Rider and Driver  : 
     - Both rider and driver see each other’s real-time location on the map during the journey.
   
   -   Backend Workflow  :
     -   Location Sharing  : Using WebSockets, both the driver and rider exchange location data at regular intervals, which the platform updates in real time on the user interface.
     -   Database Persistence  : Key locations (start, end, checkpoints) are saved in PostgreSQL for ride history, while the MongoDB database stores granular location logs to enhance future analysis.

  # 5.   Completing the Ride  
   -   Driver  :
     - Upon reaching the destination, the driver marks the ride as "completed."
   
   -   Backend Workflow  :
     -   Fare Calculation  : Based on ride distance, time, and other factors, the backend calculates the fare.
     -   Payment Processing  : If integrated with a payment gateway, the rider’s registered payment method is charged.
     -   Record Keeping  :
       - Ride summary and fare are saved in PostgreSQL.
       - MongoDB logs the trip details and user interactions for historical reference and analytics.

  # 6.   Viewing Ride History (Rider’s Perspective)  
   -   Rider  :
     - Can view their previous rides, including pickup/drop-off points, distance, time, fare, and driver ratings.
   
   -   Backend Workflow  :
     -   MongoDB Query  : The system queries MongoDB to fetch ride logs and user activity records.
     -   REST API  : The frontend retrieves this data and displays it in the rider’s ride history interface.

  # 7.   System Monitoring and Maintenance  
   -   Admin/Backend Team  :
     - Real-time dashboards track platform performance, such as active rides, server load, driver availability, and Kafka queue processing times.
     - Alerts are set up to monitor for any latency in Redis or downtime in WebSocket connections.

---

  #   End-to-End Ride Demo Summary  
   -   Rider Requests a Ride   → Kafka queues the request → Redis matches the rider to a driver → WebSocket streams real-time locations.
   -   Driver Accepts Ride   → WebSocket provides live updates between rider and driver → PostgreSQL records ride data.
   -   Ride Completes   → Fare calculated, payment processed, and data archived for both PostgreSQL (structured) and MongoDB (activity logs).
