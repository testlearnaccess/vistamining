# RMPOBIL0 - Input Dependencies and Outputs Analysis

## INPUT DEPENDENCIES

### Required Global Variables (Must be Set)
- **RMPOXITE** - Home Oxygen Contract Site/Location ID
- **RMPODATE** - Billing month date (format: YYMM00)
- **RMPORVDT** - Billing period date (derived from RMPODATE)
- **RMPOVDR** - Vendor ID for billing transactions
- **QUIT** - Control flag for early termination

### FileMan Database Files (Read Dependencies)

#### Primary Files
- **^RMPO(665.72)** - Home Oxygen Billing file
  - Structure: Site → Month → Vendor → Patient → Items
  - Contains billing transaction records

- **^RMPR(665)** - Prosthetics Patient file
  - Subfiles accessed:
    - `"RMPOA"` - Home oxygen patient data (activation, inactivation, site)
    - `"RMPOB"` - Prescription data
    - `"RMPOC"` - Item/equipment data
  - Index: `"AHO"` - Activation date cross-reference

- **^DPT(n,.35)** - Patient Demographics file
  - Field .35: Date of Death
  - Used for deceased patient validation

#### FileMan Data Dictionary Files
- **^DD(665.72,1,0)** - Billing file field definitions
- **^DD(665.723,1,0)** - Vendor subfile field definitions  
- **^DD(665.7231,9,0)** - Patient subfile field definitions
- **^DD(665.72319,1,0)** - Item subfile field definitions

### Interactive User Inputs

#### MAIN Entry Point
- **Site Selection** (via HOSITE^RMPOUTL0)
- **Billing Month** (via MONTH routine with DIC call)
- **Vendor Selection** (via VENDOR routine with DIC call)

#### PREBILL Entry Point
- **Site Selection** (via HOSITE^RMPOUTL0)  
- **Billing Month** (via DIR call with date validation)

### External Routine Dependencies
- **%ZIS** - Device/terminal control
- **RMPOUTL0** - Prosthetics utility routines
- **RMPOLM** - List manager interface
- **RMPOPST3** - Post-processing routines
- **RMPOPED** - Prosthetics editing routines
- **^DIC** - FileMan data input control
- **^DIE** - FileMan data editing
- **^DIR** - FileMan directory/prompting
- **^%DT** - Date/time utilities
- **^DIP** - FileMan print/report generator
- **XUSCLEAN** - VistA cleanup utilities

## OUTPUTS

### Database Records Created/Updated

#### Billing File Structure (^RMPO(665.72))
```
Site Level: ^RMPO(665.72,SITE,0)
  ↓
Month Level: ^RMPO(665.72,SITE,1,MONTH,0)
  ↓  
Vendor Level: ^RMPO(665.72,SITE,1,MONTH,1,VENDOR,0)
  - Field 1: Creation timestamp (set to NOW)
  ↓
Patient Level: ^RMPO(665.72,SITE,1,MONTH,1,VENDOR,"V",PATIENT,0)
  ↓
Item Level: ^RMPO(665.72,SITE,1,MONTH,1,VENDOR,"V",PATIENT,1,ITEM_IEN,0)
```

#### Item Record Fields Created
- **Field 1**: Primary Item indicator
- **Field 2**: HCPCS code (medical procedure code)
- **Field 3**: Fund Control Point  
- **Field 4**: Remarks/Notes
- **Field 5**: Unit Cost
- **Field 6**: Total Cost (Quantity × Unit Cost)
- **Field 7**: Quantity
- **Field 9**: ICD-9 diagnosis code
- **Field 12**: Original item IEN reference
- **Field 13**: Item Type
- **Field 14**: Unit of Issue

### User Interface Outputs

#### Screen Displays
- **Progress Messages**: "Generating [MONTH] billing transactions for [VENDOR]"
- **Status Messages**: "This may take a while..."
- **Error Messages**: Via BLDSTAT function for patient exclusions

#### Interactive Prompts
- Site selection menus
- Month selection with date validation
- Vendor selection from available vendors
- "Press Enter to Continue" prompts

### Reports Generated

#### Pre-billing Report (PREBILL entry)
- **Template**: [RMPO-BILLING-PRESORT]
- **Sort Order**: [RMPO-BILLING-PRESORT]
- **Filter Logic**: Patient validation via OK2BLD
- **Output**: Patient status report showing inclusion/exclusion reasons

#### Status Codes for Patient Processing
- **"OK"** - Patient processed successfully
- **"Different Home Oxygen Contract Location"** (Error -1)
- **"No Home Oxygen Information"** (Error -2)  
- **"Deactivated"** (Error -3)
- **"No RX on file"** (Error -4, -5)
- **"RX expires prior to billing period"** (Error -6)
- **"No items on file"** (Error -7)
- **"No items for vendor"** (Error -8)
- **"Patient deceased, not inactivated"** (Error -9)

### System State Changes

#### Global Variables Set/Modified
- **RMPODATE** - Billing date established
- **RMPORVDT** - Billing period date
- **RMPOVDR** - Selected vendor ID
- **RMPOXITE** - Selected site ID
- **RMPOMTH** - Month display format
- **RMPODFN** - Current patient being processed
- **RMPORX** - Latest prescription number
- **RMPOTOT** - Calculated item total cost

#### Cleanup Actions
- All working variables cleared via KILL^XUSCLEAN
- FileMan variables (DIC, DIE, DIR, DA, DR, etc.) cleaned up

### Error Handling Outputs

#### Validation Results
- Returns numeric codes for different exclusion reasons
- Provides descriptive text via BLDSTAT function
- Handles DTOUT, DUOUT, DIROUT timeout conditions

#### Transaction Integrity
- Prevents duplicate record creation through existence checks
- Ensures proper hierarchical record structure
- Maintains referential integrity between patient and item records

## PROCESSING SUMMARY

**Input Flow**: User selections → Database validation → Patient processing → Item calculations
**Output Flow**: Billing records → Status reports → User notifications → System cleanup

The routine transforms patient prosthetic equipment data into structured billing transactions while maintaining comprehensive audit trails and validation checks throughout the process.
