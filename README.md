# Money Laundering Detection

A comprehensive data pipeline implementing the FaSTM∀N (Fast Streaming Transaction Model for Anti-Money Laundering Networks) framework for detecting suspicious transaction patterns and money laundering activities in financial networks.

## Dataset

https://www.kaggle.com/datasets/ealtman2019/ibm-transactions-for-anti-money-laundering-aml?resource=download

## Pipeline Architecture

### 1. Temporal View Graph Construction

**Nodes**: Each transaction becomes a node with attributes:
- `node_id`: Unique transaction identifier
- `f_i`: Sender account (From)
- `b_i`: Receiver account (Beneficiary)
- `t_i`: Transaction timestamp
- `a_i`: Transaction amount
- `Is Laundering`: Ground truth label

**Edges**: Connects consecutive transactions where:
- The receiver of transaction A becomes the sender of transaction B
- Transaction B occurs within the timedelta after transaction A
- Represents the flow of money through intermediary accounts

### 2. Second-Order Graph Transformation

Transforms the temporal graph into a second-order representation where:
- Each node represents a complete transaction (sender → receiver)
- Edges represent sequential money flows between transaction pairs
- Edge weights calculated using conditional probabilities

### 3. Community Detection with Leiden Algorithm

- **Algorithm**: Leiden community detection
- **Objective**: Modularity optimization
- **Weight Filter**: Edges with weight ≥ 0.5
- **Purpose**: Group related transactions into communities that may represent laundering networks

### 4. Feature Engineering

**Community-Level Features**:
- `transactions_count`: Number of transactions in community
- `total_amount`: Total money transferred
- `average_amount`: Average transaction amount
- `active_duration`: Time span between first and last transaction
- `num_edges`: Number of connections within community
- `avg_edge_weight`: Average strength of connections
- `total_tx_in_degree`: Total incoming transaction degree
- `total_tx_out_degree`: Total outgoing transaction degree
- `degree_imbalance`: Absolute difference between in/out degrees
- `edge_density`: Ratio of edges to transactions
- `community_avg_hold_time`: Average time money is held before being transferred

**Account-Level Features**:
- `avg_hold_time`: Average duration an account holds money before transferring
- `num_hold_events`: Number of money holding events
- Community involvement patterns

### 5. Rule-Based Anomaly Detection

Five behavioral flags identify suspicious communities:

| Flag | Criteria | Weight | Suspicious Pattern |
|------|----------|--------|-------------------|
| `low_hold_time_flag` | Hold time < 10th percentile | 2 | Rapid money movement (layering) |
| `high_edge_density_flag` | Edge density > 90th percentile | 2 | Highly interconnected network |
| `short_active_duration_flag` | Active duration < 10th percentile | 1 | Burst activity pattern |
| `high_amount_flag` | Average amount > 90th percentile | 1 | Large transactions |
| `high_degree_imbalance_flag` | Degree imbalance > 90th percentile | 1 | Unbalanced flow pattern |

## Key Algorithms & Methods

### Leiden Community Detection
- **Library**: `igraph`
- **Parameters**: 
  - Directed graph
  - Weight-based modularity optimization
  - Minimum edge weight: 0.5

### Hold Time Calculation
Measures how long money stays in an account between receiving and sending:
1. Creates events for each money IN and OUT transaction
2. Sorts by account and timestamp
3. Calculates duration between consecutive IN→OUT events
4. Aggregates per account and community

### Results
- Community features DataFrame with risk scores
- Suspicious community classifications
- Account-level risk indicators

## Detection Patterns

The pipeline identifies money laundering patterns including:

1. **Layering**: Rapid movement through multiple accounts (low hold time)
2. **Structuring**: Complex transaction networks (high edge density)
3. **Smurfing**: Burst activity in short timeframes
4. **Large Transfers**: Above-average transaction amounts
5. **Flow Anomalies**: Unbalanced in/out transaction patterns

## References

- FaSTM∀N Framework: Fast Streaming Transaction Model for Anti-Money Laundering Networks
- Leiden Algorithm: Traag, V.A., Waltman, L. & van Eck, N.J. (2019)
