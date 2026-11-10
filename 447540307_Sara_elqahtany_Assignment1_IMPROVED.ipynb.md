# VNS Implementation Fixes for CFLP

## Root Cause Analysis

the Neighborhood Search (VNS) algorithm for the Capacitated Facility Location Problem (CFLP) was experiencing high optimality gaps (>15–20%) due to three main issues:

1. **Broken $N_2$ Neighborhood (`n2_close_open_facility`):** 
   Immediatelly after closing a facility, the function was reopening another one ($j_{\text{close}} = 1$). This turned $N_2$ into a duplicate of the swap neighborhood ($N_3$), making it impossible for the algorithm to reduce the total number of open facilities and cut fixed costs.
2. **Weak Shaking Step:** 
   The `shaking` step only applied a single 1-step move. Because `local_search` uses the exact same 1-step neighborhood operators, it immediately reversed the shake and returned to the same local minimum.
3. **Suboptimal Client Assignment Order:** 
   Assigning clients sequentially ($0, 1, 2, \dots$) allowing small-demand clients to take up nearby facility capacity, forcing high-demand clients onto distant facilities.

---

## Corrected Implementation Code

### 1. Pure Facility Closing Neighborhood ($N_2$)

This  updated version removes the accidental facility reopening step so $N_2$ strictly closes the  open facilities 

```python
def n2_close_open_facility(max_neighbors: int, facility_opened: np.ndarray, service_costs: np.ndarray,
                           facility_capacities: np.ndarray, client_demands: np.ndarray) -> List[Tuple[np.ndarray, np.ndarray]]:
    neighbors = []
    opened_indices = np.where(facility_opened == 1)[0]
    
    # Need at least 2 open facilities to close one
    if len(opened_indices) <= 1:
        return neighbors
    
    np.random.shuffle(opened_indices)
    
    for j_close in opened_indices:
        if max_neighbors > 0 and len(neighbors) >= max_neighbors:
            break
        
        neighbor_facility_opened = facility_opened.copy()
        neighbor_facility_opened[j_close] = 0  # Purely close the facility
        
        neighbor_client_assignments = assign_clients_to_nearest_open_facility(
            service_costs, neighbor_facility_opened, facility_capacities, client_demands
        )
        
        if is_feasible(neighbor_facility_opened, neighbor_client_assignments, facility_capacities, client_demands):
            neighbors.append((neighbor_facility_opened, neighbor_client_assignments))

    return neighbors
```

---

### 2. Demand-Sorted Client Assignment

Sorting clients by demand in descending order gives large demand clients priority access to open capacity nearby and so on

```python
def assign_clients_to_nearest_open_facility(service_costs: np.ndarray, facility_opened: np.ndarray,
                                            facility_capacities: np.ndarray, client_demands: np.ndarray) -> np.ndarray:
    num_clients = service_costs.shape[0]
    opened_indices = np.where(facility_opened == 1)[0]
    
    if len(opened_indices) == 0:
        raise ValueError("No facilities are open!")
    
    client_assignments = np.full(num_clients, -1, dtype=int)
    remaining_capacity = facility_capacities.copy()
    costs_to_open = service_costs[:, opened_indices]
    
    # Process largest demand clients first
    client_order = np.argsort(-client_demands)
    
    for client_idx in client_order:
        costs = costs_to_open[client_idx]
        sorted_fac_positions = np.argsort(costs)
        
        assigned = False
        for pos in sorted_fac_positions:
            fac_idx = opened_indices[pos]
            if remaining_capacity[fac_idx] >= client_demands[client_idx] - 1e-6:
                client_assignments[client_idx] = fac_idx
                remaining_capacity[fac_idx] -= client_demands[client_idx]
                assigned = True
                break
        
        if not assigned:
            nearest_fac = opened_indices[sorted_fac_positions[0]]
            client_assignments[client_idx] = nearest_fac
    
    return client_assignments
```

---

### 3. Multi-Step Shaking Procedure

Applies $k$ random perturbations to push the search out of local optima traps.

```python
def shaking(current_solution: Tuple[np.ndarray, np.ndarray], k: int, 
            service_costs: np.ndarray, facility_capacities: np.ndarray, 
            client_demands: np.ndarray) -> Tuple[np.ndarray, np.ndarray]:
    facility_opened, client_assignments = copy_solution(current_solution[0], current_solution[1])
    num_facilities = len(facility_opened)
    
    for _ in range(k):
        opened_indices = np.where(facility_opened == 1)[0]
        closed_indices = np.where(facility_opened == 0)[0]
        move_type = np.random.choice(['toggle', 'swap', 'reassign'])
        
        if move_type == 'toggle':
            idx = np.random.randint(num_facilities)
            if facility_opened[idx] == 1 and len(opened_indices) > 1:
                facility_opened[idx] = 0
            elif facility_opened[idx] == 0:
                facility_opened[idx] = 1
        elif move_type == 'swap' and len(opened_indices) > 0 and len(closed_indices) > 0:
            open_idx = np.random.choice(opened_indices)
            close_idx = np.random.choice(closed_indices)
            facility_opened[open_idx] = 0
            facility_opened[close_idx] = 1
        elif move_type == 'reassign' and len(client_assignments) > 0:
            c_idx = np.random.randint(len(client_assignments))
            if len(opened_indices) > 1:
                alt_facs = opened_indices[opened_indices != client_assignments[c_idx]]
                client_assignments[c_idx] = np.random.choice(alt_facs)

    if np.sum(facility_opened) == 0:
        facility_opened[np.random.randint(num_facilities)] = 1
        
    shaken_assignments = assign_clients_to_nearest_open_facility(
        service_costs, facility_opened, facility_capacities, client_demands
    )
    
    return (facility_opened, shaken_assignments)
```

---

### 4. Main VNS Algorithm (`vns_solve_cflp`)

Updateing  shaking intensity dynamically, resets to $k = 1$ when a new best solution is found, and increments $k \to k_{\max}$ when stuck.

```python
def vns_solve_cflp(instance: dict, max_iterations: int = 100, max_no_improve: int = 20, 
                   max_neighbors: int = 15, neighborhood_structures: List[str] = ['n1', 'n2', 'n3', 'n4'],
                   k_max: int = 4) -> Tuple[Tuple[np.ndarray, np.ndarray], Tuple[List[float], List[int]]]:
    
    current_solution = greedy_initial_solution(
        num_clients=instance["num_clients"], 
        num_facilities=instance["num_facilities"], 
        service_costs=instance["service_costs"],
        facility_capacities=instance["facility_capacities"],
        client_demands=instance["client_demands"]
    )
    
    current_solution = local_search(
        current_solution=current_solution,
        service_costs=instance["service_costs"],
        fixed_costs=instance["fixed_costs"],
        facility_capacities=instance["facility_capacities"],
        client_demands=instance["client_demands"],
        neighborhood_structures=neighborhood_structures,
        max_neighbors=max_neighbors
    )
    
    current_cost = compute_total_cost(
        instance["fixed_costs"], instance["service_costs"], 
        current_solution[0], current_solution[1]
    )
    
    best_solution = current_solution
    best_cost = current_cost
    trajectory = [best_cost]
    trajectory_iterations = [0]
    
    iteration = 0
    no_improve_count = 0
    k = 1
    
    while iteration < max_iterations and no_improve_count < max_no_improve:
        iteration += 1
        
        # Shake with intensity k
        shaken_solution = shaking(
            best_solution, k, instance["service_costs"],
            instance["facility_capacities"], instance["client_demands"]
        )
        
        # Local search on the shaken solution
        candidate_solution = local_search(
            current_solution=shaken_solution,
            service_costs=instance["service_costs"],
            fixed_costs=instance["fixed_costs"],
            facility_capacities=instance["facility_capacities"],
            client_demands=instance["client_demands"],
            neighborhood_structures=neighborhood_structures,
            max_neighbors=max_neighbors
        )
        
        candidate_cost = compute_total_cost(
            instance["fixed_costs"], instance["service_costs"], 
            candidate_solution[0], candidate_solution[1]
        )
        
        # Neighborhood change rule
        if candidate_cost < best_cost - 1e-5:
            best_solution = candidate_solution
            best_cost = candidate_cost
            trajectory.append(best_cost)
            trajectory_iterations.append(iteration)
            no_improve_count = 0
            k = 1  # Found better solution -> drop back to light shaking
        else:
            no_improve_count += 1
            k = (k % k_max) + 1  # Stuck -> increase shake intensity
            
    return best_solution, (trajectory, trajectory_iterations)
```