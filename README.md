# Tableau Sales & Customer Dashboard Project 

## Overview
This tutorial demonstrates building **two professional Tableau dashboards** (Sales & Customer) from scratch, following industry best practices from Mercedes-Benz. The project covers the complete lifecycle: requirements analysis, mockup design, data preparation, chart building, and dashboard assembly.

## Problem Statement
**Goal:** Create interactive dashboards to analyze sales performance and customer behavior

**Deliverables:**
1. Sales Dashboard - Year-over-year sales metrics and trends
2. Customer Dashboard - Customer distribution and top performers

**Data Source:** Kaggle dataset (400K+ events, 5 tables)

---

## PROJECT PHASES

### The 5-Phase Approach (Like Building a House)

**Phase 1: Requirements Analysis**
- Collect and analyze user requirements
- Decide on chart types for each requirement
- Draw dashboard mockups
- Choose color schemes

**Phase 2: Data Source Preparation**
- Connect data
- Build data model
- Understand data structure

**Phase 3: Chart Building**
- Create calculated fields
- Test calculations
- Build charts
- Format visuals (colors, axes, tooltips)

**Phase 4: Dashboard Structure Planning**
- Plan container hierarchy
- Build foundations (containers)
- Add content to containers

**Phase 5: Final Touches**
- Add filters and interactivity
- Insert icons and navigation
- Test thoroughly

---

## SALES DASHBOARD REQUIREMENTS

### User Story

**Purpose:** Provide overview of sales metrics and trends to analyze year-over-year performance

### Key Requirements

**1. KPI Overview**
- Display total sales, profit, quantity for current year
- Compare with previous year
- Show percentage difference

**Chart Type:** BANs (Big Ass Numbers)

**2. Sales Trends**
- Monthly data for sales, profit, quantity
- Compare current year vs. previous year
- Highlight highest and lowest months

**Chart Type:** Sparkline charts

**3. Product Subcategory Comparison**
- Compare sales by subcategory (current vs. previous year)
- Include profit comparison
- Show performance indicators

**Chart Type:** Bar-in-bar charts

**4. Weekly Trends**
- Weekly sales and profit data
- Show average lines
- Highlight above/below average weeks

**Chart Type:** Line charts with area fill

### Interactivity Requirements

**Dynamic Year Selection:**
- Users select any year (not just current)
- Use parameters for flexibility

**Navigation:**
- Buttons to switch between dashboards
- Easy dashboard navigation

**Chart Filters:**
- Click charts to filter dashboard
- Interactive data exploration

**Data Filters:**
- Product filters (category, subcategory)
- Location filters (region, state, city)
- Toggle show/hide for space saving

---

## DASHBOARD MOCKUP DESIGN

### Color Scheme (4-Color System)

**Basic Colors:**
1. **Dark Gray:** #2D3A3A (text, emphasis)
2. **Light Gray:** #E8E8E8 (backgrounds, secondary elements)

**Accent Colors:**
3. **Turquoise:** #08CFDC (positive, profit, max values)
4. **Orange:** #FF6B35 (negative, loss, min values)

**Why 4 colors?**
- Maintains consistency
- Professional appearance
- Easy to interpret
- Reduces visual clutter

### Layout Structure

```
┌─────────────────────────────────────────────────┐
│ LOGO | Sales Dashboard 2023 | 🏪📊🔍          │
├─────────────────────────────────────────────────┤
│ Legend: ━ 2023  ━ 2022  ⚫Max ⚫Min             │
├────────────────┬────────────────┬───────────────┤
│ Total Sales    │ Total Profit   │ Total Quantity│
│ $X.XXM         │ $XXX.XXK       │ XXX.XK        │
│ ↑ XX.X%        │ ↓ XX.X%        │ ↑ XX.X%       │
│ [Sparkline]    │ [Sparkline]    │ [Sparkline]   │
├────────────────┴────────────────┴───────────────┤
│ Sales & Profit by Subcategory                   │
│ [Bar-in-bar chart] | [Profit bars]              │
├─────────────────────────────────────────────────┤
│ Weekly Trends                                   │
│ [Sales line chart] | [Profit line chart]        │
└─────────────────────────────────────────────────┘
```

---

## DATA PREPARATION

### Dataset Structure (5 Tables)

**1. Orders (Fact Table)**
- order_id, customer_id, product_id
- order_date, ship_date
- sales, quantity, profit

**2. Customers (Dimension)**
- customer_id, customer_name

**3. Products (Dimension)**
- product_id, category, subcategory, product_name

**4. Locations (Dimension)**
- postal_code, city, state, country, region

**5. Additional fields used**
- Segments, shipping modes

### Data Model (Star Schema)

```
        Products
            ↓
Customers → Orders ← Locations
```

**Relationships:**
- Customers.customer_id = Orders.customer_id
- Products.product_id = Orders.product_id
- Locations.postal_code = Orders.postal_code

### Data Quality Checks

**1. Check data types:**
```
✓ Dates are DATE type (not string)
✓ Numbers are NUMBER type (not string)
✓ Geographic fields have correct roles
```

**2. Rename data source:**
```
From: "orders.csv"
To: "Orders"
```

**3. Explore data:**
- Check unique values per field
- Understand hierarchies
- Identify relationships

---

## KEY CALCULATED FIELDS

### 1. Year Selection Parameter

```
Name: Select Year
Data Type: Integer
Allowable Values: List
Values from: Year(Order Date)
Current Value: 2023
```

**Purpose:** Let users choose which year is "current"

### 2. Current Year Sales

```
Name: Current Year Sales

IF YEAR([Order Date]) = [Select Year]
THEN [Sales]
ELSE NULL
END
```

### 3. Previous Year Sales

```
Name: Previous Year Sales

IF YEAR([Order Date]) = [Select Year] - 1
THEN [Sales]
ELSE NULL
END
```

### 4. Sales Percent Difference

```
Name: Percent Difference Sales

(SUM([Current Year Sales]) - SUM([Previous Year Sales])) 
/ SUM([Previous Year Sales])
```

**Format:** Percentage with 1 decimal

**Display Format (with KPI symbols):**
```
Positive: ↑ XX.X%
Negative: ↓ XX.X%
```

### 5. Min/Max Sales for Sparklines

```
Name: Min Max Sales

IF SUM([Current Year Sales]) = WINDOW_MAX(SUM([Current Year Sales]))
THEN SUM([Current Year Sales])
ELSEIF SUM([Current Year Sales]) = WINDOW_MIN(SUM([Current Year Sales]))
THEN SUM([Current Year Sales])
END
```

**Purpose:** Highlight highest/lowest months in sparklines

### 6. KPI: Current Year Below Previous

```
Name: KPI Current Year Less Than Previous Year

IF SUM([Current Year Sales]) < SUM([Previous Year Sales])
THEN "⚠"  // Warning circle
ELSE ""
END
```

**Purpose:** Show warning when current year underperforms

### 7. Sales Average KPI (for Weekly Trends)

```
Name: KPI Sales Average

IF SUM([Current Year Sales]) > WINDOW_AVG(SUM([Current Year Sales]))
THEN "Above"
ELSE "Below"
END
```

**Purpose:** Color weeks above/below average

### Complete Calculation Set

**For each metric (Sales, Profit, Quantity), create:**
1. Current Year [Metric]
2. Previous Year [Metric]
3. Percent Difference [Metric]
4. Min Max [Metric]

**Total:** ~15-20 calculated fields

---

## CHART BUILDING DETAILS

### Chart 1: KPI BAN with Sparkline

**Structure:**
- Title contains BAN (Big numbers)
- Chart area contains sparkline

**Title Content:**
```
Total Sales                    (Font: 14pt, Gray)
$2.30M                        (Font: 22pt, Bold, Dark Gray)
↑ 20.4% vs previous year      (Font: 20pt, Semi-bold, KPI format)
```

**Title Setup:**
1. Double-click title
2. Insert fields:
   - Text: "Total Sales"
   - Insert > Current Year Sales
   - Text: "vs previous year"
   - Insert > Percent Difference Sales
3. Format each element separately

**Sparkline Setup:**

**Columns:** `MONTH(Order Date)` (continuous)
**Rows:** 
- `SUM(Current Year Sales)`
- `SUM(Previous Year Sales)`
- `SUM(Min Max Sales)`

**Mark Types:**
- Current/Previous Year: Line
- Min Max: Circle

**Procedure:**
1. Add month to columns (switch to continuous)
2. Add current year sales to rows
3. Add previous year sales to rows
4. Use Measure Names/Values for dual axis
5. Add Min Max sales
6. Change Min Max mark type to Circle
7. Right-click on Min Max axis → Dual Axis
8. Synchronize axes
9. Hide right axis

**Formatting:**
- Remove all grid lines
- Remove row/column dividers
- Show only Jan and Dec on x-axis
- Colors: Dark gray (current), light gray (previous)
- Min/Max circles: Turquoise (max), Orange (min), 70% opacity

**Axis Trick to Fix Title:**
- Add brackets to axis labels: `[value]`
- This prevents interaction with sparkline
- Then replace in title with unbracketed version

**Tooltip Configuration:**
```
Sales of [Month] [Current Year]: [Current Year Sales]
Sales of [Month] [Previous Year]: [Previous Year Sales]
Sales Difference: [Percent Difference Sales]
Highest/Lowest Sales: [Min Max Sales]
```

**Format numbers consistently:**
- Right-click field → Default Properties → Number Format
- Custom: `$#,###K` (thousands with dollar sign)

---

### Chart 2: Subcategory Comparison (Bar-in-Bar)

**Structure:** Two side-by-side bar charts

**Left Side - Sales Comparison:**
- Background bar: Previous Year Sales
- Foreground bar: Current Year Sales

**Right Side - Profit:**
- Single bar colored by profit (positive/negative)

**Setup:**

**Rows:** `Subcategory`
**Columns:** 
- `SUM(Previous Year Sales)`
- `SUM(Current Year Sales)`
- `SUM(Current Year Profit)`

**Bar-in-Bar Steps:**
1. Add previous year sales to columns
2. Add current year sales to columns
3. Reduce size of current year bar
4. Color: Previous (light gray), Current (dark gray)
5. Right-click current year → Dual Axis
6. Synchronize axes
7. Hide second axis

**Profit Coloring:**
1. Hold CTRL, drag profit to Color
2. Edit Colors → 2-step gradient
3. Low (negative): Orange
4. High (positive): Turquoise

**Add KPI Warning Symbol:**
1. Add calculated field: `KPI Current Year Less Than Previous Year`
2. Place before subcategory name
3. Format: Orange color, Tableau Medium font

**Sorting:**
- Right-click subcategory → Sort
- Field: Current Year Sales
- Order: Descending

---

### Chart 3: Weekly Trends

**Structure:** Two separate line charts (sales and profit)

**Setup:**

**Columns:** `WEEK(Order Date)` (continuous)
**Rows:** 
- `SUM(Current Year Sales)`
- `SUM(Current Year Profit)`

**Why two separate charts?**
- Each needs its own average reference line
- Easier to read than dual axis with two averages

**Add Reference Lines:**
1. Right-click on Sales axis → Add Reference Line
2. Value: Average
3. Label: Custom → `Average <Value>`
4. Format: Dashed line, 40% opacity
5. Repeat for Profit

**Color by Performance:**
1. Add `KPI Sales Average` to Color (Sales chart)
2. Add `KPI Profit Average` to Color (Profit chart)
3. Edit Colors:
   - Above: Turquoise
   - Below: Orange

**Path Type:**
- Change from Linear to Step
- Gives more modern appearance

**Formatting:**
- Remove grid lines (keep row dividers to separate charts)
- Format axes with dollar signs
- Consistent number formats

---

## DASHBOARD ASSEMBLY

### Container Structure (Nested Hierarchy)

```
Main Vertical Container
├── Horizontal Container (Title)
│   ├── Logo Image
│   ├── Title Text
│   └── Horizontal Container (Buttons)
│       ├── Sales Button
│       └── Customer Button
│
├── Legend Sheet (sparkline legend)
│
├── Horizontal Container (KPIs)
│   ├── KPI Sales
│   ├── KPI Profit
│   └── KPI Quantity
│
└── Horizontal Container (Charts)
    ├── Vertical Container (Chart 1)
    │   ├── Title Text
    │   ├── Horizontal Container (Legends)
    │   │   ├── Legend Sheet
    │   │   └── Profit Legend Text
    │   └── Subcategory Chart
    │
    └── Vertical Container (Chart 2)
        ├── Title Text
        ├── Legend Text
        └── Weekly Trends Chart

Filter Vertical Container (Floating)
├── Blank (spacing)
├── "FILTERS" Title Text
├── Blank (spacing)
├── "PRODUCT" Text
├── Category Filter
├── Subcategory Filter
├── Blank (spacing)
├── "LOCATION" Text
├── Region Filter
├── State Filter
└── City Filter
```

### Step-by-Step Assembly

**Step 1: Create Main Container**

1. Create new dashboard
2. Set size: Fixed 1200px × 800px
3. Drag vertical container to canvas (main container)
4. Add border (temporary - for visibility): Orange
5. Rename: "Main Container"

**Step 2: Create Filter Container**

1. Drag any worksheet to canvas (creates auto container)
2. Hold SHIFT, click floating icon
3. Remove worksheet
4. Add border: Purple (temporary)
5. Rename: "Filter Container"

**Step 3: Build Title Container**

1. Drag horizontal container into main container
2. Add border: Blue (temporary)
3. Rename: "Title Container"
4. Add Image object (logo)
5. Add Text object (title)
6. Add Navigation objects (2 buttons)
7. Nest buttons in mini horizontal container
8. Distribute contents evenly

**Step 4: Add Legend**

1. Place legend sheet between title and KPIs
2. Remove borders

**Step 5: Build KPI Container**

1. Drag horizontal container below title
2. Add border: Blue (temporary)
3. Rename: "KPI Container"
4. Add 3 blank placeholders
5. Replace blanks with KPI sheets
6. Distribute contents evenly
7. Change backgrounds to white

**Step 6: Build Charts Container**

1. Drag horizontal container below KPIs
2. Add border: Blue (temporary)
3. Rename: "Charts Container"
4. For each chart:
   - Create vertical sub-container
   - Add title text
   - Add legend (horizontal container for multiple legends)
   - Add chart sheet
   - Set background to white

**Step 7: Remove Temporary Borders**

- Remove all colored borders from containers
- Set dashboard background to light gray (#F5F5F5)

### Spacing and Padding Rules

**Outer Padding (Between Charts):**
- Standard: 10px all sides
- Top of KPIs: 0px (no space above legend)
- Creates 20px gaps (10px + 10px from neighbor)

**Inner Padding (Inside Containers):**
- Standard: 7px all sides
- Gives breathing room
- Keeps content from edges

**How to Apply:**

1. Select chart/container
2. Go to Layout tab
3. Outer Padding: 10
4. Uncheck "All sides equal"
5. Set top to 0 (for KPIs)
6. Inner Padding: 7

**Result:** Clean, professional spacing throughout

---

## FILTERS AND INTERACTIVITY

### Filter Setup

**Required Filters:**
1. Category (Product)
2. Subcategory (Product)
3. Region (Location)
4. State (Location)
5. City (Location)

**Apply to All Worksheets:**

1. Add filter to any worksheet
2. Right-click filter → Apply to Worksheets → All Using This Data Source
3. Small icon appears indicating global filter

**Filter Container Design:**

**Structure:**
```
[Blank - spacing]
FILTERS  (14pt, White, Bold, Centered, Letter-spaced)
[Blank - spacing]
P R O D U C T  (11pt, Light Gray, Centered, Letter-spaced)
  Category ▼
  Subcategory ▼
[Blank - spacing]
L O C A T I O N  (11pt, Light Gray, Centered, Letter-spaced)
  Region ▼
  State ▼
  City ▼
```

**Filter Format:**
- Type: Multiple Values (Dropdown)
- Shows compact list
- Remove "All" option for low-cardinality fields

**Container Styling:**
- Background: Dark gray (#2D3A3A)
- Inner padding: 10px
- Position: Right edge, full height

### Show/Hide Filter Container

**Button Configuration:**

1. Select filter container
2. More Options → Add Show/Hide Button
3. Button appears (small icon)
4. Right-click button → Edit Button
5. When hidden: Filter icon
6. When shown: X icon
7. Tooltips: "Show Dashboard Filters" / "Close Dashboard Filters"

**Icon Upload:**
- Choose image → Navigate to project/icons folder
- Select appropriate icon file

### Chart Interactivity

**Use as Filter:**

1. Click chart on dashboard
2. Click filter icon (funnel)
3. Chart becomes clickable filter
4. Clicking elements filters other charts

**Which charts to make filterable:**
- Subcategory bars
- Weekly trends
- ❌ KPI BANs (usually not needed)

**Test:** Click bar → Other charts update

---

## NAVIGATION AND ICONS

### Navigation Buttons

**Sales Dashboard Button:**
- Navigate to: Sales Dashboard
- Image (hidden): Sales icon (active - colored)
- Image (shown): Sales icon (inactive - gray)
- Tooltip: "Go to Sales Dashboard"

**Customer Dashboard Button:**
- Navigate to: Customer Dashboard  
- Image (hidden): Customer icon (inactive - gray)
- Image (shown): Customer icon (active - colored)
- Tooltip: "Go to Customer Dashboard"

**Button Container:**
- Nest both buttons in horizontal container
- Distribute evenly for equal sizing
- Convert to floating
- Position at top-right

### Icons Used

**Files (in project/icons folder):**
1. `filter-icon.png` - Filter/search symbol
2. `close-icon.png` - X symbol
3. `sales-active.png` - Colored sales chart icon
4. `sales-inactive.png` - Gray sales chart icon
5. `customer-active.png` - Colored customer icon
6. `customer-inactive.png` - Gray customer icon
7. `logo.png` - Company logo

**Adding Icons:**

1. Select button/object
2. Edit Button
3. Choose Image
4. Navigate to file
5. Select OK
6. Repeat for alternate state

### Logo Placement

1. Drag Image object to title container
2. Place before title text
3. Choose logo file
4. Center and Fit Entire View
5. Adjust container sizes for balance

---

## CUSTOMER DASHBOARD

### Requirements

**1. KPI Overview**
- Total Customers
- Sales per Customer
- Total Orders
- Compare current vs. previous year

**2. Trends**
- Monthly customer/sales/order trends
- Sparklines with min/max

**3. Customer Distribution**
- Histogram showing customers by order count

**4. Top 10 Customers**
- Ranked by profit
- Show: Rank, Name, Last Order, Profit, Sales, Orders

### Key Calculated Fields

**1. Current Year Customers**

```
Name: Current Year Customers

IF YEAR([Order Date]) = [Select Year]
THEN [Customer ID]
ELSE NULL
END
```

**2. Sales per Customer**

```
Name: Current Year Sales per Customer

SUM([Current Year Sales]) / COUNTD([Current Year Customers])
```

**3. Customer Order Count (LOD Expression)**

```
Name: Customer Order Count

{FIXED [Current Year Customers] : COUNTD([Current Year Order ID])}
```

**Why LOD?** 
- Creates bins for histogram
- Counts orders per customer
- Fixed LOD doesn't change with view level

**Test LOD:** Always verify in table:
```
Rows: Customer Name, Order ID, Order Date
Columns: Customer Order Count

Result: Should show count of orders per customer
```

**4. Top 10 Rank**

```
Simply use: INDEX()

Convert to Discrete
Place at start of rows
```

### Chart: Histogram

**Setup:**

**Columns:** `Customer Order Count` (as dimension)
**Rows:** `COUNTD(Current Year Customers)`

**Formatting:**
- Mark type: Bar
- Color: Turquoise
- Borders: White, visible
- Remove grid lines
- Add labels

**Result:** Distribution showing how many customers placed 1, 2, 3... orders

### Chart: Top 10 Table

**Setup:**

**Rows:** 
- `INDEX()` (rank)
- `Customer Name`

**Columns (left to right):**
- Text (for names and rank)
- `SUM(Current Year Profit)`
- `SUM(Current Year Sales)`
- `COUNTD(Current Year Order ID)`
- `MAX(Order Date)`

**Filter:**
1. Add Customer Name to filters
2. Top 10 by Field
3. Field: Current Year Profit
4. Aggregation: Sum
5. Order: Descending

**Formatting:**

**Rank Column:**
- Prefix: `#`
- Background: Light gray
- Font: Bold

**Headers:**
- Create custom header row using horizontal container
- Text objects for each column
- Hide original headers
- Align text boxes with columns

**Custom Headers:**
```
#  |  Customer  |  Last Order  |  [Year] Profit  |  [Year] Sales  |  Orders
```

**Styling:**
- Alternate row colors (minimal)
- Remove grid lines
- Dark text
- Clean, readable font

---

## DASHBOARD ASSEMBLY SHORTCUTS

### Duplicate Approach

Instead of building customer dashboard from scratch:

1. Right-click Sales Dashboard → Duplicate
2. Rename to "Customer Dashboard"
3. Replace charts one by one:
   - Drag new chart onto old chart
   - Old chart is replaced
   - Container structure preserved
4. Update titles
5. Update legends
6. Remove unnecessary filters from floating container

**Benefits:**
- Maintains spacing
- Preserves structure
- Much faster
- Consistent design

### Common Tasks

**Change All Sheets to "Fit Entire View":**
- Click each sheet on dashboard
- Dropdown → Fit → Entire View
- Ensures optimal space usage

**Test Structure:**
- Open Layout pane
- Check item hierarchy
- Verify nesting is correct
- Rename containers meaningfully

**Remove Old Content:**
- Delete old sheets
- Remove unused containers
- Clear floating objects

---

## PROFESSIONAL FORMATTING CHECKLIST

### Colors
- 4-color system throughout
- Dark gray for emphasis
- Light gray for secondary
- Turquoise for positive
- Orange for negative
- Background: Lightest gray

### Typography
- Consistent fonts (Tableau Book, Medium, Semi-bold)
- Hierarchy: Titles (14pt), Values (22pt), Details (10pt)
- Letter-spacing for headers
- Proper alignment

### Spacing
- 20px between charts (10px + 10px)
- 7px inner padding
- 0px top padding for KPIs
- White backgrounds on cards

### Charts
- No grid lines
- No unnecessary headers
- Hidden axis titles (put in chart title instead)
- Custom tooltips
- Formatted numbers ($, K, %)

### Interactivity
- Parameter control for year
- Charts usable as filters
- Product and location filters
- Show/hide filter panel
- Navigation buttons

### Icons & Navigation
- Logo in header
- Active/inactive button states
- Filter icon with toggle
- Tooltips on all buttons
- Smooth navigation between dashboards

---

## TESTING CHECKLIST

### Functionality Tests

**Parameter Testing:**
```
1. Change year to 2022
   ✓ All KPI values update
   ✓ All charts update
   ✓ Sparklines show correct year
   ✓ Percent differences recalculate

2. Change year to 2021
   ✓ Same checks as above

3. Return to 2023
   ✓ Verify return to original state
```

**Filter Testing:**
```
1. Select Category: Furniture
   ✓ All charts filter
   ✓ KPIs recalculate
   ✓ Subcategories show only Furniture
   
2. Add Region: East
   ✓ Further filtering applied
   ✓ No errors
   
3. Clear all filters
   ✓ Return to full dataset
```

**Interactivity Testing:**
```
1. Click subcategory bar
   ✓ Other charts filter
   ✓ Filter highlight visible
   
2. Click week on trend chart
   ✓ Dashboard filters
   
3. Click outside to deselect
   ✓ Filters clear
```

**Navigation Testing:**
```
1. Click Customer Dashboard button
   ✓ Navigate to Customer Dashboard
   ✓ Button shows as active
   ✓ Sales button shows as inactive
   
2. Click Sales Dashboard button
   ✓ Return to Sales Dashboard
   ✓ Button states swap
```

**Filter Panel Testing:**
```
1. Click filter icon
   ✓ Panel slides in from right
   ✓ Icon changes to X
   
2. Apply filters
   ✓ Filters work correctly
   
3. Click X icon
   ✓ Panel hides
   ✓ Icon changes back to filter
   ✓ Filters remain applied
```

### Visual Tests

**Alignment:**
- All KPIs same height
- Charts aligned with container edges
- Text properly aligned
- Icons evenly spaced

**Spacing:**
- Consistent gaps between elements
- No overlapping content
- Proper padding inside containers
- Clean edges

**Responsiveness:**
- Adjust dashboard size
- Check if elements resize properly
- Verify text doesn't overflow

---

## COMMON ISSUES & SOLUTIONS

### Issue 1: BAN Not Updating

**Problem:** Sparkline changes but BAN stays same

**Cause:** Using fixed field in title instead of parameter-based field

**Solution:**
```
Wrong: Insert > Order Date > Year
Right: Insert > Select Year Parameter
```

### Issue 2: Sparkline Breaks When Adding to Dashboard

**Problem:** BAN shows as range (e.g., "2023 (0)")

**Cause:** Month field is continuous, creates axis

**Solution:**
```
1. Double-click axis label
2. Add brackets: [value] at start and end
3. Go into title
4. Remove bracketed field
5. Re-insert unbracketed version
```

### Issue 3: Bar-in-Bar Not Aligned

**Problem:** Bars don't overlay correctly

**Cause:** Axes not synchronized

**Solution:**
```
1. Right-click second axis
2. "Synchronize Axis"
3. Both bars now use same scale
```

### Issue 4: Spacing Inconsistent

**Problem:** Gaps between charts look uneven

**Cause:** Different padding values

**Solution:**
```
1. Select each chart/container
2. Layout > Outer Padding: 10
3. Uncheck "All sides equal"
4. Top: 0 (for KPIs only)
5. Inner Padding: 7 (all charts)
```

### Issue 5: Filter Doesn't Affect All Charts

**Problem:** Filter only works on one worksheet

**Cause:** Filter not applied globally

**Solution:**
```
1. Right-click filter
2. Apply to Worksheets
3. Select "All Using This Data Source"
4. Verify icon appears on filter
```

### Issue 6: LOD Calculation Returns Wrong Values

**Problem:** Customer order count shows unexpected numbers

**Cause:** Not using correct granularity

**Solution:**
```
Test in table view:
Rows: Customer Name, Order ID
Columns: Your LOD calculation

Verify count matches visible orders
```

### Issue 7: Dashboard Looks Cluttered

**Problem:** Too much information, hard to read

**Solutions:**
- Remove unnecessary axis labels
- Hide redundant headers
- Use white backgrounds on charts
- Increase spacing (10px → 15px)
- Reduce font sizes slightly
- Remove grid lines everywhere
- Use filter panel instead of always-visible filters

---

## BEST PRACTICES LEARNED

### Requirements Phase

1. **Always start with mockup** before building
2. **Choose colors early** and stick to them
3. **Get user approval** on mockup before coding
4. **Document requirements** clearly
5. **Choose appropriate chart types** for data

### Data Preparation

1. **Check data quality** immediately
2. **Rename everything** for clarity
3. **Verify relationships** in data model
4. **Test data** before building charts
5. **Create year extraction field** for parameters

### Calculation Creation

1. **Build incrementally** (current year, then previous, then difference)
2. **Test in table view** before adding to charts
3. **Use consistent naming** (Current Year [Metric])
4. **Format numbers** at field level (default properties)
5. **Test LOD calculations** thoroughly

### Chart Building

1. **Build chart first**, format later
2. **Remove clutter** (grids, headers, excess labels)
3. **Use dual axis** carefully (always synchronize)
4. **Test with parameter** at each step
5. **Custom tooltips** for better UX

### Dashboard Assembly

1. **Plan container structure** before dragging sheets
2. **Name all containers** meaningfully
3. **Build foundations first** (containers)
4. **Add content second** (charts)
5. **Format last** (spacing, colors)
6. **Don't rush** - take time to plan

### Spacing & Layout

1. **Decide on spacing value** (e.g., 20px) and stick to it
2. **Outer padding:** Half the desired gap (10px → 20px total)
3. **Inner padding:** Consistent (7px everywhere)
4. **Use blanks** for custom spacing
5. **Test alignment** with colored borders (remove after)

### Filters & Interactivity

1. **Apply filters globally** from start
2. **Use dropdown filters** for space
3. **Hide filter panel** by default
4. **Make primary charts filterable** (not KPIs)
5. **Test all filter combinations**

### Professional Finish

1. **Consistent formatting** across all dashboards
2. **Icons for navigation** (not text buttons)
3. **Tooltips on everything** clickable
4. **Remove all temporary borders**
5. **Test thoroughly** before presenting

---

## KEY FORMULAS REFERENCE

### Parameter Setup
```
Name: Select Year
Type: Integer
Values: List from YEAR([Order Date])
Current: 2023
```

### Current Year Pattern
```
IF YEAR([Order Date]) = [Select Year]
THEN [Metric]
ELSE NULL
END
```

### Previous Year Pattern
```
IF YEAR([Order Date]) = [Select Year] - 1
THEN [Metric]
ELSE NULL
END
```

### Percent Difference
```
(SUM([Current Year Metric]) - SUM([Previous Year Metric])) 
/ SUM([Previous Year Metric])
```

### Min/Max for Sparklines
```
IF SUM([Current Year Metric]) = WINDOW_MAX(SUM([Current Year Metric]))
THEN SUM([Current Year Metric])
ELSEIF SUM([Current Year Metric]) = WINDOW_MIN(SUM([Current Year Metric]))
THEN SUM([Current Year Metric])
END
```

### Above/Below Average
```
IF SUM([Current Year Metric]) > WINDOW_AVG(SUM([Current Year Metric]))
THEN "Above"
ELSE "Below"
END
```

### KPI Warning Symbol
```
IF SUM([Current Year Metric]) < SUM([Previous Year Metric])
THEN "⚠"
ELSE ""
END
```

### Customer Order Count (LOD)
```
{FIXED [Current Year Customers] : COUNTD([Current Year Orders])}
```

### Sales per Customer
```
SUM([Current Year Sales]) / COUNTD([Current Year Customers])
```

---

## EXCEL FORMULAS

**Note:** This tutorial uses **Tableau Desktop**, not Excel. No Excel formulas are involved.

**However, equivalent Excel concepts:**

**Year Extraction:**
```excel
=YEAR(A2)
```

**Conditional Sum (Current Year):**
```excel
=SUMIF(Year_Column, 2023, Sales_Column)
```

**Percent Difference:**
```excel
=(Current - Previous) / Previous
```

**Rank:**
```excel
=RANK(A2, $A$2:$A$11, 0)
```

**Count Orders per Customer:**
```excel
=COUNTIFS(Customer_Column, A2, Year_Column, 2023)
```

---

## PROJECT DELIVERABLES

### Files Created

**Tableau Workbook (.twbx):**
- Sales Dashboard
- Customer Dashboard
- 20+ worksheets
- 15+ calculated fields
- 5+ parameters

**Charts Built:**
1. KPI Sales (BAN + Sparkline)
2. KPI Profit (BAN + Sparkline)
3. KPI Quantity (BAN + Sparkline)
4. Subcategory Comparison (Bar-in-bar + Profit bars)
5. Weekly Trends (2 line charts with reference lines)
6. KPI Customers (BAN + Sparkline)
7. KPI Sales per Customer (BAN + Sparkline)
8. KPI Orders (BAN + Sparkline)
9. Customer Distribution (Histogram)
10. Top 10 Customers (Detail table)

**Supporting Elements:**
- Custom legends (3)
- Filter panel (collapsible)
- Navigation buttons (2)
- Logo image
- Filter/close icons

### Time Investment

**Phase 1 (Requirements & Mockup):** 1-2 hours
**Phase 2 (Data Preparation):** 30 minutes
**Phase 3 (Chart Building):** 4-6 hours
**Phase 4 (Dashboard Assembly):** 2-3 hours
**Phase 5 (Final Touches):** 1-2 hours

**Total:** 8-14 hours for complete project

**With practice:** 4-6 hours possible

---

## REAL-WORLD APPLICATION

### Use Cases

**Sales Teams:**
- Track performance vs. targets
- Identify underperforming products
- Spot trends early
- Make data-driven decisions

**Management:**
- Executive overview (KPIs)
- Year-over-year comparison
- Resource allocation
- Strategic planning

**Marketing:**
- Understand customer behavior
- Identify top customers
- Target high-value segments
- Campaign effectiveness

**Operations:**
- Monitor order patterns
- Weekly performance tracking
- Identify operational issues
- Capacity planning

### Industry Applications

**Retail:**
- Store performance dashboards
- Product category analysis
- Customer segmentation
- Seasonal trend analysis

**E-commerce:**
- Online sales tracking
- Customer lifetime value
- Cart abandonment analysis
- Channel performance

**Manufacturing:**
- Production KPIs
- Quality metrics
- Supplier performance
- Inventory turnover

**Finance:**
- Revenue dashboards
- Profitability analysis
- Budget vs. actual
- Forecast tracking

**Healthcare:**
- Patient volume metrics
- Department performance
- Cost analysis
- Quality indicators

---

## NEXT STEPS & EXTENSIONS

### Enhancements

**1. Add More Metrics:**
- Gross margin percentage
- Customer acquisition cost
- Average order value
- Return rate

**2. Advanced Calculations:**
- Moving averages
- Year-to-date (YTD) totals
- Growth rates
- Cohort analysis

**3. Additional Visualizations:**
- Geographic map (by state/city)
- Product hierarchy drilldown
- Customer journey funnel
- Scatter plot (sales vs. profit)

**4. Forecasting:**
- Trend lines
- Predictive analytics
- What-if scenarios
- Goal tracking

**5. Advanced Interactivity:**
- Dashboard actions (filter, highlight, URL)
- Parameter actions (dynamic selection)
- Set actions (dynamic grouping)
- Sheet swapping

### Publishing Options

**Tableau Public:**
- Free hosting
- Public dashboards
- Portfolio showcase
- Limited to 15GB

**Tableau Server:**
- Enterprise solution
- Secure, private
- User management
- Scheduled refreshes

**Tableau Online:**
- Cloud-based
- Same as Server
- No infrastructure
- Subscription model

**Export Options:**
- PDF (static)
- PowerPoint (static)
- Image (PNG)
- Data (CSV/Excel)

---

## CONCLUSION

### What Was Accomplished

**Technical Skills:**
- Requirements analysis and mockup design
- Data modeling (star schema with 5 tables)
- 15+ calculated fields with parameters
- LOD expressions for complex aggregations
- 10 different chart types
- 2 complete interactive dashboards
- Custom container structure
- Professional formatting and spacing
- Filter panel with show/hide
- Navigation between dashboards

**Business Value:**
- Sales performance tracking
- Year-over-year comparison
- Product category analysis
- Customer behavior insights
- Top performer identification
- Data-driven decision support

**Design Principles:**
- 4-color system for consistency
- Minimalist approach (remove clutter)
- Consistent spacing (20px between charts)
- Custom legends and tooltips
- Professional typography
- User-friendly navigation

### Key Learnings

1. **Planning is everything** - Don't rush to build
2. **Structure first, content second** - Build containers first
3. **Test calculations in tables** - Before adding to charts
4. **Consistent formatting** - Decide early, apply everywhere
5. **User-centric design** - Think about end-user experience
6. **Iterative approach** - Build, test, refine, repeat

### Mercedes-Benz Practices Shared

- Phase-based project approach
- Requirements documentation
- Mockup approval before development
- Incremental testing
- Container-based architecture
- Consistent design language
- Professional delivery standards

---

*This comprehensive tutorial demonstrates industry-standard practices for creating professional Tableau dashboards suitable for enterprise environments. The methodical approach ensures maintainability, scalability, and user satisfaction.*

**Note:** This tutorial contains **no Excel formulas** - it is entirely focused on Tableau Desktop development using calculated fields, LOD expressions, and dashboard design best practices.
