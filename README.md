# Stock trading engine

1. Data Structure
Orders Array: A fixed-size array (orders[MAX_ORDERS]) stores all orders. Each Order struct contains order_type, ticker, quantity, price, timestamp, and active status.

Ticker Mapping: Since dictionaries are not allowed, we map ticker symbols (e.g., "T123") to integers (0-1023) using a simple hash function (ticker_to_index).

Atomic Counters: std::atomic<int> for order_count and next_order_id ensures lock-free updates to shared state.

2. Concurrency
Lock-Free Updates: We use std::atomic for next_order_id and order_count. The compare_exchange_weak operation ensures thread-safe slot allocation in addOrder.

Order Matching: The matchOrder function avoids locks by using careful ordering of updates. Quantities are updated directly, and active flags are toggled atomically via memory ordering.

Memory Ordering: We use std::memory_order_acq_rel and std::memory_order_release to ensure proper visibility of updates across threads.

3. addOrder Function
Takes order_type, ticker_symbol, quantity, and price.

Converts the ticker symbol to an integer index using a hash function.

Allocates a slot in the orders array using atomic operations (compare_exchange_weak).

Adds the order and calls matchOrder.

4. matchOrder Function
Runs in O(n) time by scanning all orders in the array.

For a new buy order, finds the lowest-priced sell order with a matching ticker that satisfies the price condition.

For a new sell order, finds the highest-priced buy order.

Uses price-time priority: prefers the best price; for equal prices, prefers the earliest timestamp.

Updates quantities and deactivates orders when fully matched, using careful ordering to avoid races.

5. Simulation
The simulateActiveTrading function launches multiple threads, each adding random orders.

Randomly generates order types, ticker symbols (e.g., "T123"), quantities, and prices.

Ensures concurrent access to the order book.

6. Constraints Met
No Dictionaries/Maps: Only arrays and basic structs are used.

O(n) Time Complexity: Matching scans all orders in O(n) time.

Lock-Free: Uses std::atomic for shared state updates, avoiding locks.

Supports 1024 Tickers: Tickers are mapped to integers (0-1023).

Simulation: Random orders are generated across multiple threads.

