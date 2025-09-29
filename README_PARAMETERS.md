# YOLO Dataset Creator - Parameter Update Guide

## Overview
This guide explains how to update product parameters that appear in the tagging interface. Parameters are stored in Supabase and loaded dynamically by the web application.

---

## System Architecture

```
פרמטרים.xlsx (Excel File)
    ↓
parameters_loader.py (Python Script)
    ↓
parameters.csv (CSV Export)
    ↓
Supabase Database (parameters table)
    ↓
index2.html (Web Application)
```

---

## How to Update Parameters

### Method 1: Edit Existing Parameters in Supabase (Quick)

**Use this when:** You want to change a parameter name, min/max values, or defaults

1. Login to Supabase: https://supabase.com
2. Select your project (`kwoiohzrzmwkxkihkogb`)
3. Go to **Table Editor** → **parameters** table
4. Click on any row to edit:
   - `param_name` - Hebrew display name
   - `min_value`, `max_value` - Slider range
   - `default_value` - Starting value
   - `step_value` - Slider increment
5. Click "Save"
6. **Done!** All users will see changes immediately (no code deployment needed)

---

### Method 2: Add/Remove Parameters via Excel (Complete Update)

**Use this when:** You need to add new parameters, remove parameters, or add new product types

#### Step 1: Edit the Excel File

1. Open `פרמטרים.xlsx`
2. Follow the structure:
   - **Column A:** `Product` (apple, tomato, cucumber, avocado, mango, chicken)
   - **Column B:** `Type` (visible or hidden)
   - **Columns C-N:** Parameter pairs (ID and Name)
     - `Param_1_ID`, `Param_1_Name`
     - `Param_2_ID`, `Param_2_Name`
     - ... up to `Param_6_ID`, `Param_6_Name`

**Example Row:**
```
Product: apple
Type: visible
Param_1_ID: color_quality
Param_1_Name: איכות צבע
Param_2_ID: shape_integrity
Param_2_Name: תקינות צורה
```

#### Step 2: Generate CSV

Run the Python script to convert Excel to CSV:

```bash
python parameters_loader.py
```

This creates `parameters.csv` with proper encoding.

**Alternative:** Manually save Excel as CSV (UTF-8 encoding)

#### Step 3: Clear Old Data in Supabase

In Supabase SQL Editor, run:

```sql
DELETE FROM parameters;
```

This removes all existing parameters.

#### Step 4: Import New Data

**Option A: CSV Import**
1. Go to **Table Editor** → **parameters**
2. Click **"Insert"** → **"Import data via spreadsheet"**
3. Upload `parameters.csv`
4. Click **"Import"**

**Option B: SQL Import** (Better for Hebrew)
1. Go to **SQL Editor**
2. Create INSERT statements from your CSV
3. Run the query

Example SQL:
```sql
INSERT INTO parameters (product, param_type, param_id, param_name, min_value, max_value, default_value, step_value) VALUES
('apple', 'visible', 'color_quality', 'איכות צבע', 0, 10, 5, 0.5),
('apple', 'visible', 'shape_integrity', 'תקינות צורה', 0, 10, 5, 0.5);
-- ... more rows
```

#### Step 5: Verify

1. Check the `parameters` table in Supabase
2. Confirm Hebrew text displays correctly
3. Test the web application: reload `index2.html` and select a product category

---

## Adding a New Product Type

To add a new product (e.g., "banana"):

### 1. Update Excel File
Add rows for the new product with visible and hidden parameters

### 2. Update HTML Categories
Edit `index2.html` and add a new category button:

```html
<button class="category-btn" data-category="6" data-product="banana">🍌 בננה</button>
```

And update the arrays:
```javascript
const categories = ['avocado', 'mango', 'cucumber', 'apple', 'tomato', 'chicken_breast', 'banana'];
const categoryNames = ['אבוקדו', 'מנגו', 'מלפפון', 'תפוח', 'עגבניה', 'חזה עוף', 'בננה'];
```

### 3. Follow Method 2 steps to import parameters

---

## Database Schema Reference

### `parameters` Table Structure

| Column | Type | Description |
|--------|------|-------------|
| id | int8 | Primary key (auto-increment) |
| product | text | Product identifier (apple, mango, etc) |
| param_type | text | "visible" or "hidden" |
| param_id | text | Parameter identifier (color_quality, etc) |
| param_name | text | Hebrew display name |
| min_value | float8 | Minimum slider value |
| max_value | float8 | Maximum slider value |
| default_value | float8 | Default slider value |
| step_value | float8 | Slider increment |
| created_at | timestamptz | Timestamp (auto) |

---

## Troubleshooting

### Hebrew Characters Display as Gibberish
- **Cause:** Wrong CSV encoding
- **Solution:** Save CSV as UTF-8, or use SQL INSERT method instead

### Parameters Don't Load in Web App
- **Cause:** Browser cache or JavaScript error
- **Solution:** 
  1. Hard refresh: Ctrl+F5 (Windows) or Cmd+Shift+R (Mac)
  2. Check browser console for errors (F12)
  3. Verify Supabase connection in status bar

### Wrong Number of Parameters
- **Cause:** Duplicate or missing rows in database
- **Solution:** 
  1. Check total row count in Supabase (should be 8 rows per product: 4 visible + 4 hidden)
  2. Use SQL to verify: `SELECT product, param_type, COUNT(*) FROM parameters GROUP BY product, param_type;`

---

## Best Practices

1. **Always test changes** on `index2.html` before replacing `index.html`
2. **Keep backups** of `פרמטרים.xlsx` with version numbers
3. **Document changes** - add a changelog to this README
4. **Coordinate updates** - inform team members before making changes
5. **Verify Hebrew encoding** - always check parameter names display correctly

---

## Files in This Repository

- `index.html` - Original tagger (simple version)
- `index2.html` - Enhanced tagger with dynamic parameters
- `פרמטרים.xlsx` - Parameter configuration (source of truth)
- `parameters_loader.py` - Python script for Excel → JSON/CSV conversion
- `parameters.csv` - Generated CSV for import
- `README_PARAMETERS.md` - This file

---

## Change Log

### 2025-01-XX - Initial Setup
- Created parameters table in Supabase
- Imported 6 products with 48 total parameters
- Deployed index2.html with dynamic loading

---

## Support

For questions or issues:
1. Check Supabase logs: **Database** → **Logs**
2. Review browser console (F12) for JavaScript errors
3. Verify table structure matches schema above

---

## Future Enhancements

Potential improvements:
- [ ] Admin panel for editing parameters without SQL
- [ ] Version control for parameter changes
- [ ] A/B testing different parameter configurations
- [ ] Export annotated data with parameter values for ML training