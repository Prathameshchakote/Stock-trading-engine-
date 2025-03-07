# Stock trading engine

# Order Matching System

This project implements a simple order matching system with concurrency control and atomic operations. The system supports adding orders, matching them based on price-time priority, and running simulations of active trading with multiple threads.

## Features

- **Order Book Storage**: Orders are stored in a fixed-size array.
- **Ticker Mapping**: Ticker symbols are mapped to integers using a hash function (0-1023).
- **Concurrency**: The system uses atomic operations to ensure lock-free updates and avoid race conditions.
- **Order Matching**: Matching of orders is done based on price-time priority, ensuring the best price is matched first and the earliest timestamp is used when prices are equal.
- **Simulation**: Multiple threads simulate active trading by randomly generating orders and processing them concurrently.

## 1. Data Structure

### Orders Array
- A fixed-size array `orders[MAX_ORDERS]` stores all orders.
- Each order is represented as a struct containing:
  - `order_type` (Buy/Sell)
  - `ticker` (Stock symbol)
  - `quantity` (Amount of stock to trade)
  - `price` (Price per unit)
  - `timestamp` (When the order was placed)
  - `active` (Flag to mark if the order is still active)

### Ticker Mapping
- Ticker symbols (e.g., "T123") are mapped to integers (0-1023) using a custom hash function `ticker_to_index`. 
- This avoids the use of dictionaries/maps, adhering to the constraints.

### Atomic Counters
- `std::atomic<int> order_count` and `std::atomic<int> next_order_id` ensure thread-safe and lock-free updates to shared state.
  
## 2. Concurrency

### Lock-Free Updates
- The system uses `std::atomic` for the `next_order_id` and `order_count` to ensure safe, concurrent updates without using traditional locks.
- The `compare_exchange_weak` operation ensures thread-safe allocation of slots in the `orders` array.

### Order Matching
- The `matchOrder` function performs order matching without locks by carefully ordering updates.
- For each order, quantities are updated directly, and the active flags are toggled atomically using memory ordering.

### Memory Ordering
- `std::memory_order_acq_rel` and `std::memory_order_release` are used to ensure proper visibility of updates across threads.

## 3. addOrder Function

The `addOrder` function is responsible for adding a new order to the system. It takes the following parameters:
- `order_type` (Buy/Sell)
- `ticker_symbol` (Ticker symbol for the stock)
- `quantity` (Amount of stock to be traded)
- `price` (Price at which the stock is to be traded)

### Steps:
1. Converts the `ticker_symbol` to an integer index using a hash function (`ticker_to_index`).
2. Allocates a slot in the `orders` array using atomic operations (`compare_exchange_weak`).
3. Adds the order and calls `matchOrder` to attempt matching it with existing orders.

## 4. matchOrder Function

The `matchOrder` function is responsible for matching orders in the order book. It operates in O(n) time, where n is the number of orders in the book.

### For a Buy Order:
- The function scans the orders array and finds the lowest-priced sell order with a matching ticker symbol that satisfies the price condition.

### For a Sell Order:
- The function finds the highest-priced buy order with a matching ticker symbol.

### Price-Time Priority:
- The system prefers the best price first. For orders with the same price, it matches based on the earliest timestamp.

### Deactivating Orders:
- Once an order is fully matched, its quantity is updated, and the order is marked as inactive.

## 5. Simulation

The `simulateActiveTrading` function simulates active trading with multiple threads. Each thread adds random orders to the order book, creating a high level of concurrency and stress testing the order matching system.

### Random Order Generation:
- The simulation randomly generates order types, ticker symbols, quantities, and prices to simulate real trading conditions.

### Concurrency:
- The function ensures multiple threads concurrently add orders to the order book, simulating real-world trading activity.

## 6. Constraints Met

- **No Dictionaries/Maps**: The system avoids using dictionaries or maps, relying on arrays and basic structs.
- **O(n) Time Complexity**: The `matchOrder` function scans all orders in O(n) time to find a matching order.
- **Lock-Free**: `std::atomic` is used for shared state updates, ensuring thread-safe operations without the need for locks.
- **Supports 1024 Tickers**: Ticker symbols are mapped to integers between 0 and 1023.
- **Simulation**: The system can simulate concurrent trading by generating random orders across multiple threads.

## Building and Running

To build and run the system:

1. Clone this repository.
2. Use a C++11 or later compatible compiler.
3. Compile the source code.
