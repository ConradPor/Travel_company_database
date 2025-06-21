CREATE DATABASE Travel_agency;
GO

USE Travel_agency;
GO
-- Customer table
-- Contains data of customers who have purchased tours. Includes personal and contact information.
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(255) NOT NULL UNIQUE,
    Phone NVARCHAR(50),
    DateOfBirth DATE,
    CHECK (LEN(FirstName) >= 2),
    CHECK (LEN(LastName) >= 2)
);

-- Sellers table
-- Stores information about employees (salesmen) responsible for selling tours.
CREATE TABLE Sellers (
    SellerID INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(255) NOT NULL UNIQUE,
    HireDate DATE NOT NULL,
    CHECK (HireDate <= GETDATE())
);

-- Table of destinations
-- Represents available travel destinations. Includes the name of the country, optional city, and a description of the trip.
CREATE TABLE Destinations (
    DestinationID INT PRIMARY KEY IDENTITY(1,1),
    Country NVARCHAR(100) NOT NULL,
    City NVARCHAR(100) NULL,
    Description NVARCHAR(MAX),
    StartDate DATE NOT NULL,
    EndDate DATE NOT NULL,
    CHECK (StartDate <= EndDate)
);

-- Flight table
-- Contains data on flights assigned to tours, including airlines, flight numbers and departure/arrival information.
CREATE TABLE Flights (
    FlightID INT IDENTITY PRIMARY KEY,
    Airline NVARCHAR(100) NOT NULL,
    FlightNumber NVARCHAR(20) NOT NULL,
    Departure DATETIME NOT NULL,
    Arrival DATETIME NOT NULL,
    DepartureAirport NVARCHAR(100) NOT NULL,
    ArrivalAirport NVARCHAR(100) NOT NULL
);

-- Hotels table
-- Stores data on hotels available for tours, including name, location and number of stars.
CREATE TABLE Hotels (
    HotelID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(255) NOT NULL,
    Address NVARCHAR(255) NOT NULL,
    City NVARCHAR(100) NOT NULL,
    Stars INT CHECK (Stars >= 1 AND Stars <= 5),
    CHECK (LEN(Name) >= 2)
);

-- Transportation table
-- Contains information about additional means of transportation used during the trip (e.g., coaches, trains).
CREATE TABLE Transport (
    TransportID INT PRIMARY KEY IDENTITY(1,1),
    Type NVARCHAR(50) NOT NULL,
    Company NVARCHAR(100),
    DepartureLocation NVARCHAR(100) NOT NULL,
    ArrivalLocation NVARCHAR(100) NOT NULL,
    DepartureDate DATETIME NOT NULL,
    ArrivalDate DATETIME NOT NULL,
    CHECK (Type IN ('Bus', 'Train', 'Ship', 'Car', 'Other'))
);
-- Sales table
-- Records tour sales: who bought, from which vendor, when and where, and total cost.
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY IDENTITY(1,1),
    CustomerID INT NOT NULL FOREIGN KEY REFERENCES Customers(CustomerID),
    SellerID INT NOT NULL FOREIGN KEY REFERENCES Sellers(SellerID),
    DestinationID INT NOT NULL FOREIGN KEY REFERENCES Destinations(DestinationID),
    SaleDate DATE NOT NULL,
    TotalAmount DECIMAL(10,2) CHECK (TotalAmount >= 0),
    CHECK (SaleDate <= GETDATE())
);

-- Table of assignment of hotels to sales
-- Link between sales and hotels. Allows multiple hotels to be assigned to a single tour, with dates of stay and order.
CREATE TABLE SaleHotels (
    SaleHotelID INT PRIMARY KEY IDENTITY(1,1),
    SaleID INT NOT NULL FOREIGN KEY REFERENCES Sales(SaleID),
    HotelID INT NOT NULL FOREIGN KEY REFERENCES Hotels(HotelID),
    CheckIn DATE NOT NULL,
    CheckOut DATE NOT NULL,
    OrderInTrip INT NOT NULL CHECK (OrderInTrip > 0),
    AssignedDate DATE NOT NULL,
    CHECK (CheckIn < CheckOut)
);
--adds a unique index for saleID and OrderInTrip so there are no errors
CREATE UNIQUE INDEX UX_SaleHotels_SaleID_OrderInTrip
ON SaleHotels(SaleID, OrderInTrip);

-- Table for assigning flights to sales
-- Link between sales and flights. Allows you to assign multiple flights to a single trip, with the order and date of assignment.
CREATE TABLE SaleFlights (
    SaleFlightID INT PRIMARY KEY IDENTITY(1,1),
    SaleID INT NOT NULL FOREIGN KEY REFERENCES Sales(SaleID),
    FlightID INT NOT NULL FOREIGN KEY REFERENCES Flights(FlightID),
    OrderInTrip INT NOT NULL CHECK (OrderInTrip > 0),
    AssignedDate DATE NOT NULL
);
--adds Unique Index for SaleID and OrderInTrip so there are no errors
CREATE UNIQUE INDEX UX_SaleFlights_SaleID_OrderInTrip
ON SaleFlights(SaleID, OrderInTrip);

-- Transportation assignment table for sales
-- Link between sales and additional transportation. Includes assignment date and order in the tour schedule.
CREATE TABLE SaleTransport (
    SaleTransportID INT PRIMARY KEY IDENTITY(1,1),
    SaleID INT NOT NULL FOREIGN KEY REFERENCES Sales(SaleID),
    TransportID INT NOT NULL FOREIGN KEY REFERENCES Transport(TransportID),
    OrderInTrip INT NOT NULL CHECK (OrderInTrip > 0),
    AssignedDate DATE NOT NULL
);
--adds a unique index for saleID and OrderInTrip so that there are no errors
CREATE UNIQUE INDEX UX_SaleTransport_SaleID_OrderInTrip
ON SaleTransport(SaleID, OrderInTrip);

VIEWS:

-- The vw_CustomerSalesSummary view shows the total number of tours and the amount spent by each customer (using aggregation).
CREATE VIEW vw_CustomerSalesSummary AS
SELECT
    C.CustomerID,
    C.FirstName,
    C.LastName,
    COUNT(S.SaleID) AS TotalTrips,
    SUM(S.TotalAmount) AS TotalSpent
FROM Customers C
LEFT JOIN Sales S ON C.CustomerID = S.CustomerID
GROUP BY C.CustomerID, C.FirstName, C.LastName;

--The vw_SaleDetailsExpanded view shows details of the sale, including customer, vendor, direction and date.
CREATE VIEW vw_SaleDetailsExpanded AS
SELECT
    S.SaleID,
    C.FirstName + ' ' + C.LastName AS CustomerName,
    SL.FirstName + ' ' + SL.LastName AS SellerName,
    D.Country,
    D.City,
    S.SaleDate,
    S.TotalAmount
FROM Sales S
JOIN Customers C ON S.CustomerID = C.CustomerID
JOIN Sellers SL ON S.SellerID = SL.SellerID
JOIN Destinations D ON S.DestinationID = D.DestinationID;

--The vw_FlightUsagePerDestination view for each flight shows which countries the flight was assigned to (and how many times).
CREATE VIEW vw_FlightUsagePerDestination AS
SELECT
    F.FlightID,
    F.Airline,
    F.FlightNumber,
    D.Country,
    COUNT(SF.SaleFlightID) AS TimesUsed
FROM Flights F
JOIN SaleFlights SF ON F.FlightID = SF.FlightID
JOIN Sales S ON SF.SaleID = S.SaleID
JOIN Destinations D ON S.DestinationID = D.DestinationID
GROUP BY F.FlightID, F.Airline, F.FlightNumber, D.Country;




--PROCEDURES

-- Procedure adds a new customer if it does not exist
CREATE PROCEDURE sp_AddCustomerIfNotExists
    @FirstName NVARCHAR(100),
    @LastName NVARCHAR(100),
    @Email NVARCHAR(255),
    @Phone NVARCHAR(50),
    @DateOfBirth DATE
AS
BEGIN
    SET NOCOUNT ON;
    IF NOT EXISTS (SELECT 1 FROM Customers WHERE Email = @Email)
    BEGIN
        INSERT INTO Customers (FirstName, LastName, Email, Phone, DateOfBirth)
        VALUES (@FirstName, @LastName, @Email, @Phone, @DateOfBirth);
    END
    ELSE
    BEGIN
        PRINT 'Customer with this email already exists.';
    END
END;

-- Procedure updates the tour price by the specified amount (can be positive or negative)
CREATE PROCEDURE sp_UpdateSalePrice
    @SaleID INT,
    @AmountChange DECIMAL(10,2),
    @SellerID INT
AS
BEGIN
    SET NOCOUNT ON;

    -- Setting the session context for SellerID
    EXEC sys.sp_set_session_context @key = N'SellerID', @value = @SellerID;

    -- Checking whether sales exist
    IF EXISTS (SELECT 1 FROM Sales WHERE SaleID = @SaleID)
    BEGIN
        -- Update price
        UPDATE Sales
        SET TotalAmount = TotalAmount + @AmountChange
        WHERE SaleID = @SaleID;

        -- Returning an updated entry

        SELECT * FROM Sales WHERE SaleID = @SaleID;
    END
    ELSE
    BEGIN
        RAISERROR('No sale found with the given ID.', 16, 1);
    END
END;
FUNCTIONS

-- Returns a list of hotels in a given city with a given number of stars
CREATE FUNCTION fn_GetHotelsByCityAndStars(
    @City NVARCHAR(100),
    @Stars INT
)
RETURNS TABLE
AS
RETURN
    SELECT HotelID, Name, Address, Stars
    FROM Hotels
    WHERE City = @City AND Stars = @Stars;


-- A function that returns all tours of a customer with the given name and email
CREATE FUNCTION fn_GetCustomerSalesByNameAndEmail
(
    @LastName NVARCHAR(100),
    @Email NVARCHAR(255)
)
RETURNS TABLE
AS
RETURN
(
    SELECT
        S.SaleID,
        S.SaleDate,
        S.TotalAmount,
        D.Country,
        D.City
    FROM Customers C
    JOIN Sales S ON C.CustomerID = S.CustomerID
    JOIN Destinations D ON S.DestinationID = D.DestinationID
    WHERE C.LastName = @LastName
      AND C.Email = @Email
);

TRIGGERS

--Checks whether the check-in date agrees with the date of the organized transportation agrees with the date of the tour
CREATE TRIGGER trg_CheckTransportDatesWithinDestination
ON SaleTransport
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF EXISTS (
        SELECT 1
        FROM INSERTED i
        JOIN Transport t ON i.TransportID = t.TransportID
        JOIN Sales s ON i.SaleID = s.SaleID
        JOIN Destinations d ON s.DestinationID = d.DestinationID
        WHERE (t.DepartureDate < d.StartDate OR t.ArrivalDate > d.EndDate)
    )
    BEGIN
        RAISERROR('Transport dates must be within the destination date range.', 16, 1);
        ROLLBACK;
    END
END;

-- Trigger logging price changing
--Create log table
CREATE TABLE SalePriceHistory (
    HistoryID INT IDENTITY(1,1) PRIMARY KEY,
    SaleID INT NOT NULL,
    OldAmount DECIMAL(10,2),
    NewAmount DECIMAL(10,2),
    ChangeDate DATETIME NOT NULL DEFAULT GETDATE(),
    ChangedBySellerID INT NOT NULL,
    FOREIGN KEY (ChangedBySellerID) REFERENCES Sellers(SellerID)
);

CREATE TRIGGER trg_LogSalePriceChange
ON Sales
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    IF EXISTS (
        SELECT 1
        FROM INSERTED i
        JOIN DELETED d ON i.SaleID = d.SaleID
        WHERE i.TotalAmount <> d.TotalAmount
    )
    BEGIN
        INSERT INTO SalePriceHistory (SaleID, OldAmount, NewAmount, ChangedBySellerID)
        SELECT
            i.SaleID,
            d.TotalAmount,
            i.TotalAmount,
            CONVERT(INT, SESSION_CONTEXT(N'SellerID'))  -- ID seller from session context
        FROM INSERTED i
        JOIN DELETED d ON i.SaleID = d.SaleID
        WHERE i.TotalAmount <> d.TotalAmount;
    END
END;
