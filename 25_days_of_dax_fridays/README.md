# 25 Days of DAX Fridays
The 25 days of DAX Fridays challenge is a Curbal initiative that consists of solving 25 DAX questions. You can find all the details [here](https://curbal.com/25-days-of-dax-fridays-challenge).

<br />


## Notes
- I used a double underscore prefix for variables and the CamelCase naming convention. I used long variable names (but no more than 35 characters) to make sure they’re meaningful. 
- Whenever a question/bonus question (I added some bonus questions) asks for an answer of type text, I used CONCATENATEX to return ties if they’re present.
- All the answers return the desired value(s) on a card visual.
- The dataset is a live dataset. Your answers might be different from my answers in the images below.

<br />
<br />


## Days 1-5 | Products
![](https://github.com/shadfrigui/dax/blob/616e80203676855884bf163275eb85600f0e0214/25_days_of_dax_fridays/files/screenshot_1.png)
#### Day 1

```
Day 1 =
-- How many current products cost less than $20?
CALCULATE (
    COUNTROWS ( Products ),
    Products[Discontinued] = FALSE ()
    	&& Products[UnitPrice] < 20
)
```

#### Day 2
```
Day 2 =
-- Which product is the most/least expensive?
-- The following code returns both most and least expensive products and handles ties.
VAR __MostExpensiveProducts =
    TOPN ( 1, Products, Products[UnitPrice], DESC )
VAR __MostExpensiveProductNames =
    CONCATENATEX (
        __MostExpensiveProducts,
        Products[ProductName],
        ", ",
        Products[ProductName], ASC
    )
VAR __LeastExpensiveProducts =
    TOPN ( 1, Products, Products[UnitPrice], ASC )
VAR __LeastExpensiveProductNames =
    CONCATENATEX (
        __LeastExpensiveProducts,
        Products[ProductName],
        ", ",
        Products[ProductName], ASC
    )
VAR __Result = 
    -- The following delimiter (open quote + Press Enter to start on a new line + close quote")
    -- ensures that least expensive products starts on a new line.
    "Most Expensive Products: "  & __MostExpensiveProductNames & "
" & "Least Expensive Products: " & __LeastExpensiveProductNames
RETURN
    __Result
```

#### Day 3
```
Day 3 =
-- What is the average unit price for our products?
AVERAGEX ( Products, Products[UnitPrice] )
```

#### Day 4
```
Day 4 =
-- How many products are above the average unit price?
VAR __AverageUnitPrice = [Day 3]
VAR __Result =
    CALCULATE ( 
        COUNTROWS ( Products ), 
        Products[UnitPrice] > __AverageUnitPrice
    )
RETURN
    __Result
```

#### Day 5
```
Day 5 =
-- How many products cost between $15 and $25 inclusive?
CALCULATE (
    COUNTROWS ( Products ),
    Products[UnitPrice] >= 15
        && Products[UnitPrice] <= 25
)
```

<br />
<br />


## Days 6-10 | Orders
![](https://github.com/shadfrigui/dax/blob/616e80203676855884bf163275eb85600f0e0214/25_days_of_dax_fridays/files/screenshot_2.png)
#### Day 6
```
Day 6.1 =
-- What is the average number of different products per order?
VAR __NumProductsPerOrder =
    ADDCOLUMNS (                               -- New columns are computed in the row context of ADDCOLUMNS. So we need 
        SUMMARIZE ( Orders, Orders[OrderID] ), -- to invoke context transition to generate a filter context by wrapping
        "@Number of Distinct Products", CALCULATE ( COUNT ( Orders[ProductID] ) ) -- the expression with CALCULATE.
    )
VAR __AverageNumProductsPerOrder =
    AVERAGEX ( __NumProductsPerOrder, [@Number of Distinct Products] )
RETURN
    __AverageNumProductsPerOrder
```

#### Day 6 Bonus Question
```
Day 6.2 = 
-- What is the average order size for all years and all customers?
VAR __TotalQty =
    SUMX ( Orders, Orders[Quantity] )
VAR __NumOrders =
    DISTINCTCOUNT ( Orders[OrderID] )
VAR __AverageOrderSize =
    DIVIDE ( __TotalQty, __NumOrders )
RETURN
    __AverageOrderSize
```

#### Day 7
```
Day 7 =
-- What is the order value in $ of open orders (not shipped yet)?
CALCULATE (
    [Total Sales],
    ISBLANK ( Orders[ShippedDate] )
)
```

#### Day 8
```
Day 8 =
-- How many orders are "single item" (only one product ordered)?
VAR __NumProductsPerOrder =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[OrderID] ),
        "@Number of Distinct Products", CALCULATE ( COUNT ( Orders[ProductID] ) )
    )
VAR __SingleItemOrders =
    FILTER ( __NumProductsPerOrder, [@Number of Distinct Products] = 1 )
VAR __NumSingleItemOrders =
    COUNTROWS ( __SingleItemOrders )
RETURN
    __NumSingleItemOrders
```

#### Day 9 First Method
```
Day 9.1 =
-- What is the average sale per transaction for "Romero y Tomillo"?
VAR __SalesPerCustomerNamePerOrder =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Customers[CompanyName], Orders[OrderID] ),
        "@Sales", CALCULATE ( [Total Sales] )
    )
VAR __RomeroOrders =
    FILTER (
        __SalesPerCustomerNamePerOrder,
        Customers[CompanyName] = "Romero y Tomillo"
    )
VAR __RomeroAverageSalePerOrder =
    AVERAGEX ( __RomeroOrders, [@Sales] )
RETURN
    __RomeroAverageSalePerOrder
```

#### Day 9 Second Method
```
Day 9.2 =
-- What is the average sale per transaction for "Romero y Tomillo"?
VAR __RomeroOrders =
    CALCULATETABLE (
        VALUES ( Orders[OrderID] ),
        Customers[CompanyName] = "Romero y Tomillo"
    )
VAR __RomeroAverageSalePerOrder =
    AVERAGEX ( __RomeroOrders, [Total Sales] )
RETURN
    __RomeroAverageSalePerOrder
```

#### Day 9 Third Method
```
Day 9.3 = 
-- What is the average sale per transaction for "Romero y Tomillo"?
CALCULATE (
    AVERAGEX ( VALUES ( Orders[OrderID] ), [Total Sales] ),
    Customers[CompanyName] = "Romero y Tomillo"
)
```

#### Day 10
```
Day 10 =
-- How many days since "North/South" last purchase?
VAR __NSLastPurchase =
    CALCULATE (
        LASTDATE ( Orders[OrderDate] ),
        Customers[CompanyName] = "North/South"
    )
VAR __DaysSinceNSLastPurchase =
    DATEDIFF ( __NSLastPurchase, TODAY (), DAY )
RETURN
    __DaysSinceNSLastPurchase
```

<br />
<br />


## Days 11-15 | Customers
![](https://github.com/shadfrigui/dax/blob/616e80203676855884bf163275eb85600f0e0214/25_days_of_dax_fridays/files/screenshot_3.png)
#### Day 11
```
Day 11.1 =
-- How many customers have ordered only once?
VAR __NumOrdersPerCustomer =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[CustomerID] ),
        "@Number of Orders", CALCULATE ( DISTINCTCOUNT ( Orders[OrderID] ) )
    )
VAR __SingleOrderCustomers =
    FILTER ( __NumOrdersPerCustomer, [@Number of Orders] = 1 )
VAR __NumSingleOrderCustomers =
    COUNTROWS ( __SingleOrderCustomers )
RETURN
    __NumSingleOrderCustomers
```

#### Day 11 Bonus Question
```
Day 11.2 =
-- Which customers have ordered only once?
VAR __NumOrdersPerCustomerName =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Customers[CompanyName] ),
        "@Number of Orders", CALCULATE ( DISTINCTCOUNT ( Orders[OrderID] ) )
    )
VAR __SingleOrderCustomers =
    FILTER ( __NumOrdersPerCustomerName, [@Number of Orders] = 1 )
VAR __SingleOrderCustomersNames = 
    CONCATENATEX ( __SingleOrderCustomers, Customers[CompanyName], "
"
    )
RETURN
    __SingleOrderCustomersNames
```

#### Day 12
```
Day 12 =
-- How many new customers are in the current year?
VAR __FirstPurchasePerCustomer =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[CustomerID] ),
        "@First Purchase", CALCULATE ( FIRSTDATE ( Orders[OrderDate] ) )
    )
VAR __CurrentYearNewCustomers =
    FILTER (
        __FirstPurchasePerCustomer,
        YEAR ( [@First Purchase] ) = [Current Year]
    )
VAR __NumCurrentYearNewCustomers =
    COUNTROWS ( __CurrentYearNewCustomers )
RETURN
    __NumCurrentYearNewCustomers
```

#### Day 13
```
Day 13.1 =
-- How many lost customers are in the current year?
VAR __LastPurchasePerCustomer =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[CustomerID] ),
        "@Last Purchase", CALCULATE ( LASTDATE ( Orders[OrderDate] ) )
    )
VAR __CurrentYearLostCustomers =
    FILTER (
        __LastPurchasePerCustomer,
        NOT ( ISBLANK ( [@Last Purchase] ) )
            && YEAR ( [@Last Purchase] ) < [Current Year]
    )
VAR __NumCurrentYearLostCustomers =
    COUNTROWS ( __CurrentYearLostCustomers )
RETURN
    __NumCurrentYearLostCustomers
```

#### Day 13 Bonus Question
```
Day 13.2 =
-- Which customers we lost in the current year? 
VAR __LastPurchasePerCustomerName =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Customers[CompanyName] ),
        "@Last Purchase", CALCULATE ( LASTDATE ( Orders[OrderDate] ) )
    )
VAR __CurrentYearLostCustomers =
    FILTER (
        __LastPurchasePerCustomerName,
        NOT ( ISBLANK ( [@Last Purchase] ) )
            && YEAR ( [@Last Purchase] ) < [Current Year]
    )
VAR __CurrentYearLostCustomersNames =
    CONCATENATEX (
        __CurrentYearLostCustomers,
        Customers[CompanyName],
        "
",
        Customers[CompanyName], ASC
    )
RETURN
    __CurrentYearLostCustomersNames
```

#### Day 14
```
Day 14 =
-- How many customers have never purchased Queso Cabrales?
VAR __NumAllCustomers =
    COUNTROWS ( Customers )
VAR __NumCustomersBoughtQC =
    CALCULATE (
        DISTINCTCOUNT ( Orders[CustomerID] ),
        Products[ProductName] = "Queso Cabrales"
    )
VAR __NumCustomersNeverBoughtQC = __NumAllCustomers - __NumCustomersBoughtQC
RETURN
    __NumCustomersNeverBoughtQC
```

#### Day 15
```
Day 15.1 =
-- How many customers have purchased only Queso Cabrales on a given order?
VAR __TempTable =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[CustomerID], Orders[OrderID] ),
        "@Number of Distinct Products", CALCULATE ( COUNT ( Orders[ProductID] ) ),
        "@Product ID", CALCULATE ( SELECTEDVALUE ( Orders[ProductID] ) )
    )
VAR __BoughtOnlyQCCustomers =
    FILTER ( __TempTable, [@Number of Distinct Products] = 1 && [@Product ID] = 11 )
VAR __NumCustomersBoughtOnlyQC =
    COUNTROWS ( __BoughtOnlyQCCustomers )
RETURN
    __NumCustomersBoughtOnlyQC
```

#### Day 15 Bonus Question
```
Day 15.2 =
-- Which customers have purchased only Queso Cabrales on a given order?
VAR __TempTable =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Customers[CompanyName], Orders[OrderID] ),
        "@Number of Distinct Products", CALCULATE ( COUNT ( Orders[ProductID] ) ),
        "@Product ID", CALCULATE ( SELECTEDVALUE ( Orders[ProductID] ) )
    )
VAR __BoughtOnlyQCCustomers =
    FILTER ( __TempTable, [@Number of Distinct Products] = 1 && [@Product ID] = 11 )
VAR __BoughtOnlyQCCustomersNames =
    CONCATENATEX (
        __BoughtOnlyQCCustomers,
        Customers[CompanyName],
        "
",
        Customers[CompanyName], ASC
    )
RETURN
    __BoughtOnlyQCCustomersNames
```

<br />
<br />


## Days 16-20 | Inventory
![](https://github.com/shadfrigui/dax/blob/616e80203676855884bf163275eb85600f0e0214/25_days_of_dax_fridays/files/screenshot_4.png)
#### Day 16
```
Day 16.1 = 
-- How many products are out of stock (including discontinued products)?
CALCULATE (
    COUNTROWS ( Products ),
    Products[UnitsInStock] = 0
)
```

#### Day 16 Bonus Question
```
Day 16.2 =
-- Which products are out of stock (including discontinued products)?
VAR __OutOfStockProducts =
    FILTER ( Products, Products[UnitsInStock] = 0 )
VAR __OutOfStockProductNames =
    CONCATENATEX (
        __OutOfStockProducts,
        Products[ProductName],
        "
",
        Products[ProductName], ASC
    )
RETURN
    __OutOfStockProductNames
```

#### Day 17
```
Day 17.1 = 
-- How many products need to be restocked?
CALCULATE (
    COUNTROWS ( Products ),
    Products[UnitsInStock] + Products[UnitsOnOrder] <= Products[ReorderLevel] -- UnitsOnOrder represents orders from 
        && Products[Discontinued] = FALSE()                                   -- suppliers, not orders committed to
)                                                                             -- customers. It must be added.
```

#### Day 17 Bonus Question
```
Day 17.2 =
-- Which Products need to be restocked?
VAR __NeedRestockProducts =
    FILTER (
        Products,
        Products[UnitsInStock] + Products[UnitsOnOrder] <= Products[ReorderLevel]
            && Products[Discontinued] = FALSE ()
    )
VAR __NeedReStockProductNames =
    CONCATENATEX (
        __NeedRestockProducts,
        Products[ProductName],
        "
",
        Products[ProductName], ASC
    )
RETURN
    __NeedReStockProductNames
```

#### Day 18
```
Day 18 = 
-- How many products have stocked units less than ordered units?
CALCULATE (
    COUNTROWS ( Products ),
    Products[UnitsInStock] < Products[UnitsOnOrder]
        && Products[Discontinued] = FALSE()
)
```

#### Day 19
```
Day 19.1 = 
-- What is the value of in-stock discontinued products?
CALCULATE (
    SUMX ( Products, Products[UnitsInStock] * Products[UnitPrice] ),
    Products[Discontinued] = TRUE()
)
```

#### Day 19 Bonus Question
```
Day 19.2 = 
-- What are the values of in-stock and on-order current and discontinued products?
VAR __InStockCurrentProductsValue =
    CALCULATE (
        SUMX ( Products, Products[UnitsInStock] * Products[UnitPrice] ),
        Products[Discontinued] = FALSE()
    )
VAR __OnOrderCurrentProductsValue =
    CALCULATE (
        SUMX ( Products, Products[UnitsOnOrder] * Products[UnitPrice] ),
        Products[Discontinued] = FALSE()
    )
VAR __InStockDiscontinuedProductsValue =
    CALCULATE (
        SUMX ( Products, Products[UnitsInStock] * Products[UnitPrice] ),
        Products[Discontinued] = TRUE()
    )
VAR __AllProductsValue =
    SUMX ( Products, ( Products[UnitsInStock] + Products[UnitsOnOrder] ) * Products[UnitPrice] )
VAR __Result =
        "Current In-Stock:              "      & FORMAT ( __InStockCurrentProductsValue, "Currency" )      & "
" &     "Current On-Order:            "        & FORMAT ( __OnOrderCurrentProductsValue, "Currency" )      & "
" &     "Discontinued In-Stock:       "        & FORMAT ( __InStockDiscontinuedProductsValue, "Currency" ) & "
" &     "All Products:                      "  & FORMAT ( __AllProductsValue, "Currency" )
RETURN
    __Result
```

#### Day 20
```
Day 20 =
-- Which vendor has the highest in-stock value?
VAR __InStockValueperSupplier =
    SELECTCOLUMNS (
        Suppliers,
        "@Supplier ID", Suppliers[SupplierID],
        "@Supplier Name", Suppliers[CompanyName],
        "@In-Stock Value", CALCULATE ( SUMX ( Products, Products[UnitsInStock] * Products[UnitPrice] ) )
    )
VAR __HighestInStockValueSuppliers =
    TOPN ( 1, __InStockValueperSupplier, [@In-Stock Value], DESC )
VAR __HighestInStockValueSupplierNames =
    CONCATENATEX (
        __HighestInStockValueSuppliers,
        [@Supplier Name],
        "
",
        [@Supplier Name], ASC
    )
RETURN
    __HighestInStockValueSupplierNames
```

<br />
<br />


## Days 21-25 | Employees
![](https://github.com/shadfrigui/dax/blob/616e80203676855884bf163275eb85600f0e0214/25_days_of_dax_fridays/files/screenshot_5.png)
#### Day 21
```
Day 21 =
-- What is the percentage of female employees?
VAR __NumFemaleEmployees =
    CALCULATE ( COUNTROWS ( Employees ), Employees[Gender] = "Female" )
VAR __NumAllEmployees =
    COUNTROWS ( Employees )
VAR __PercentFemaleEmployees =
    DIVIDE ( __NumFemaleEmployees, __NumAllEmployees )
RETURN
    __PercentFemaleEmployees
```

#### Day 22
```
Day 22 =
-- How many employees are 60-years-old or over?
VAR __EmployeesSixtyOrOver =                       -- I avoided calculating employees' age by dividing by 365 or 365.25 
    FILTER (                                       -- and used IF functions instead. The __EmployeesSixtyOrOver variable
        Employees,                                 -- returns employees that are 60 or over. It does not return employees 
        IF (                                       -- that are still below 60, even by 1 day.
            DATEDIFF ( Employees[BirthDate], TODAY (), YEAR ) >= 60,
            IF (
                MONTH ( Employees[BirthDate] ) < MONTH ( TODAY () ),
                TRUE,
                IF ( DAY ( Employees[BirthDate] ) <= DAY ( TODAY () ), TRUE )
            )
        )
    )
VAR __NumEmployeesSixtyOrOver =
    CALCULATE ( COUNTROWS ( Employees ), __EmployeesSixtyOrOver )
RETURN
    __NumEmployeesSixtyOrOver
```

#### Day 23 First Method
```
Day 23.1 =
-- Which employee had the highest sales in 2021?
VAR __2021SalesPerEmployee =
    CALCULATETABLE (
        ADDCOLUMNS (
            SUMMARIZE ( Orders, Orders[EmployeeID], Employees[Full Name] ),
            "@Sales", CALCULATE ( [Total Sales] )
        ),
        'Calendar'[Year] = 2021
    )
VAR __2021HighestSalesEmployees =
    TOPN ( 1, __2021SalesPerEmployee, [@Sales], DESC )
VAR __2021HighestSalesEmployeesNames =
    CONCATENATEX (
        __2021HighestSalesEmployees,
        Employees[Full Name],
        "
",
        Employees[Full Name], ASC
    )
RETURN
    __2021HighestSalesEmployeesNames
```

#### Day 23 Second Method
```
Day 23.2 =
-- Which employee had the highest sales in 2021?
VAR __2021SalesPerEmployee =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[EmployeeID], Employees[Full Name] ),
        "@Sales", CALCULATE ( [Total Sales], 'Calendar'[Year] = 2021 )
    )
VAR __2021HighestSalesEmployees =
    TOPN ( 1, __2021SalesPerEmployee, [@Sales], DESC )
VAR __2021HighestSalesEmployeesNames =
    CONCATENATEX (
        __2021HighestSalesEmployees,
        Employees[Full Name],
        "
",
        Employees[Full Name], ASC
    )
RETURN
    __2021HighestSalesEmployeesNames
```

#### Day 24
```
Day 24.1 = 
-- How many employees sold over $100K in 2021?
VAR __2021SalesPerEmployee =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[EmployeeID], Employees[Full Name] ),
        "@Sales", CALCULATE ( [Total Sales], 'Calendar'[Year] = 2021 )
    )
VAR __Over100KEmployees =
    FILTER ( __2021SalesPerEmployee, [@Sales] > 100000 )
VAR __NumOver100KEmployees =
    COUNTROWS ( __Over100KEmployees )
RETURN
    __NumOver100KEmployees
```

#### Day 24 Bonus Question
```
Day 24.2 = 
-- Which employees sold over $100K in 2021?
VAR __2021SalesPerEmployee =
    ADDCOLUMNS (
        SUMMARIZE ( Orders, Orders[EmployeeID], Employees[Full Name] ),
        "@Sales", CALCULATE ( [Total Sales], 'Calendar'[Year] = 2021 )
    )
VAR __Over100KEmployees =
    FILTER ( __2021SalesPerEmployee, [@Sales] > 100000 )
VAR __Over100KEmployeesNames =
	CONCATENATEX (
        __Over100KEmployees,
        Employees[Full Name],
        "
",
        Employees[Full Name],
        ASC
    )
RETURN
    __Over100KEmployeesNames
```

#### Day 25
```
Day 25 = 
-- How many employees got hired in 1994?
CALCULATE (
    COUNTROWS ( Employees ),
    YEAR ( Employees[HireDate] ) = 1994
)
```
