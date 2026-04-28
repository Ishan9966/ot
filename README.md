# MATLAB Optimization Programs

---

## 1. Simplex Method (Linear Programming)

```matlab
clc;
clear;

% Max Z = 3x1 + 5x2
% x1 + 2x2 <= 6
% 3x1 + 2x2 <= 12

% Initial Tableau
% [x1 x2 s1 s2 | RHS]

T = [1 2 1 0 6;
     3 2 0 1 12;
    -3 -5 0 0 0];

[m, n] = size(T);

while true
    % Step 1: Entering variable
    [min_val, pivot_col] = min(T(end,1:end-1));

    if min_val >= 0
        break;
    end

    % Step 2: Leaving variable
    ratios = inf(m-1,1);
    for i = 1:m-1
        if T(i,pivot_col) > 0
            ratios(i) = T(i,end) / T(i,pivot_col);
        end
    end

    [~, pivot_row] = min(ratios);

    % Step 3: Pivot
    pivot_element = T(pivot_row, pivot_col);
    T(pivot_row,:) = T(pivot_row,:) / pivot_element;

    for i = 1:m
        if i ~= pivot_row
            T(i,:) = T(i,:) - T(i,pivot_col) * T(pivot_row,:);
        end
    end
end

disp('Final Tableau:');
disp(T);

disp('Optimal value of Z:');
disp(T(end,end));
```

---

## 2. Transportation Problem (Minimum Cost Method)

```matlab
clc;
clear;

% Min Z = 8x11 + 6x12 + 10x13 + 9x21 + 7x22 + 4x23
% Supply: S1 = 20, S2 = 30
% Demand: D1 = 10, D2 = 25, D3 = 15

% Cost matrix
C = [8 6 10;
     9 7 4];

% Supply and Demand
supply = [20 30];
demand = [10 25 15];

[m, n] = size(C);
allocation = zeros(m,n);

while any(supply > 0) && any(demand > 0)

    % Step 1: Find minimum cost
    min_cost = inf;

    for i = 1:m
        for j = 1:n
            if supply(i) > 0 && demand(j) > 0
                if C(i,j) < min_cost
                    min_cost = C(i,j);
                    row = i;
                    col = j;
                end
            end
        end
    end

    % Step 2: Allocate
    x = min(supply(row), demand(col));
    allocation(row, col) = x;

    % Step 3: Update
    supply(row) = supply(row) - x;
    demand(col) = demand(col) - x;
end

disp('Allocation Matrix:');
disp(allocation);

total_cost = sum(sum(allocation .* C));

disp('Total Transportation Cost:');
disp(total_cost);
```

---

## 3. Big-M Method

```matlab
clc;
clear;

% Min Z = 2x1 + 3x2
% x1 + x2 >= 4
% x1 + 2x2 = 6
% x1, x2 >= 0

M = 1000;

% Tableau: x1 x2 s1 a1 a2 | RHS
% Columns: [decision vars | surplus/slack vars | artificial vars | RHS]
T = [1 1 -1 1 0 4;
     1 2  0 0 1 6;
    -2 -3  0 -M -M 0];

% art_cols: column indices of artificial variables in T
% Change this to match your tableau (e.g. [4] if only one artificial var,
% or [5 6] if artificials are in columns 5 and 6, etc.)
art_cols = [4, 5];

[m, n] = size(T);

% Fix objective row: for each constraint row whose basic variable is an
% artificial, add M * that row to eliminate the artificial from the objective
for k = 1:length(art_cols)
    for i = 1:m-1
        if T(i, art_cols(k)) == 1
            T(end,:) = T(end,:) + M * T(i,:);
        end
    end
end

max_iter = 50;
iter = 0;

while true
    iter = iter + 1;

    if iter > max_iter
        disp('Iteration limit reached');
        break;
    end

    % Entering variable
    [min_val, pivot_col] = min(T(end,1:end-1));

    if min_val >= 0
        break;
    end

    % Leaving variable
    ratios = inf(m-1,1);
    for i = 1:m-1
        if T(i,pivot_col) > 0
            ratios(i) = T(i,end) / T(i,pivot_col);
        end
    end

    [min_ratio, pivot_row] = min(ratios);

    if isinf(min_ratio)
        disp('Unbounded solution');
        break;
    end

    % Pivot
    pivot_element = T(pivot_row, pivot_col);
    T(pivot_row,:) = T(pivot_row,:) / pivot_element;

    for i = 1:m
        if i ~= pivot_row
            T(i,:) = T(i,:) - T(i,pivot_col) * T(pivot_row,:);
        end
    end
end

disp('Final Tableau:');
disp(T);

disp('Optimal value of Z:');
disp(T(end,end));
```

---

## 4. Gradient Descent

```matlab
clc;
clear;

% Min f(x1, x2) = x1^2 + x2^2 + x1*x2
% Starting point: x0 = (2, 2)
% Learning rate alpha = 0.1

% Define the objective function and its gradient as function handles.
% Change f and grad_f to solve a different problem.
f      = @(x) x(1)^2 + x(2)^2 + x(1)*x(2);
grad_f = @(x) [2*x(1) + x(2);
               2*x(2) + x(1)];

x = [2; 2];       % starting point (change as needed)
alpha = 0.1;      % learning rate
tol = 1e-5;
max_iter = 1000;

for k = 1:max_iter

    % Gradient
    grad = grad_f(x);

    % Update
    x_new = x - alpha * grad;

    if norm(x_new - x) < tol
        break;
    end

    x = x_new;
end

disp('Optimal point:');
disp(x);

disp('Minimum value:');
disp(f(x));
```