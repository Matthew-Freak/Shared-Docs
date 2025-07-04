## Changes to `tblBlindsOrderDetails`
1. Added `GuideSystem` as `short text`. Back populated with:
```SQL
UPDATE tblBlindsOrderDetails
SET GuideSystem = 'TRACK'
WHERE GuideSystem IS NULL;
```
2. Added `CableAnchor` as `short text`. Back populated with:
```SQL
UPDATE tblBlindsOrderDetails
SET CableAnchor = 'N/A'
WHERE CableAnchor IS NULL;
```
3. Added `CableSpring` as `short text`. Back populated with: 
```SQL
UPDATE tblBlindsOrderDetails
SET CableSpring = 'N/A'
WHERE CableSpring IS NULL;
```
3. Positioned Columns where appropriate

## Changes to `frmBlindsOrderDetails`

### Physical Inputs
1. Added `GuideSystem` `Combo Box` + label. Updated Tab index to 1![[Pasted image 20250627151612.png]]
2. Added `CableAnchor` `Combo Box` + label ![[Pasted image 20250704105443.png]]
3. Added `0;;;"N/A"` to Format -> Format for `SplineTapeLength`
4. Added `CableSpring` `Combo Box` ![[Pasted image 20250627153150.png]]
5. Updated tabbing

---
### Rules

---
#### Changing Records
Changing between records will directly update the **options** but not the value of the following fields:
- Cable Anchor
- Blind Type
- Spline Tape Colour
- Side Spline Tape Length - (disables / enables)
- End Plates
- Pelmet Type
- L/H Spring / Spring - (swaps field)
- Helper (spring) - (disables / enables)
- Bottom Bar Type
- Handle Type
- Weight Bars
- CL Position
- CL Flat Bar Colour

---
#### Guide System
Guide System can be set to Track, Cable or N/A. N/A is for guide system agnostic orders such as fixed panels or fabric only.

Changing the value of Guide System (`GuideSystem_AfterUpdate`) will affect the options available and **directly change / reset** the values of the following fields:
- Cable Anchor
- Blind Type
- Spline Tape Colour
- Side Spline Tape Length
- End Plates
- Pelmet Type
- L/H Spring / Spring
- Helper (spring)
- Bottom Bar Type
- Handle Type
- Weight Bars
- CL Position
- CL Flat Bar Colour

---
#### Cable Anchor type
Only available when cable guided
##### Cable Anchor Type options
Set by `LoadCableAnchorTypeOptions` 
if `GuideSystem` = `"CABLE"` then `CableAnchor` options = `"WALL FIXING", "FLOOR FIXING", "NONE"`
else options = `"N/A"` 

##### Cable Anchor Type values
Set by `SetCableAnchorValueAndOptions`
if `GuideSystem` changed to `"CABLE"` then `CableAnchor` = `"SELECT"`
else `CableAnchor` = `"N/A"`

Set by `BlindType_AfterUpdate`
if `GuideSystem` = `"CABLE"` and `BlindType` = `"FIXED PANEL", "SKIN ONLY", "FABRIC ONLY"` then `CableAnchor` = `"N/A"`
else `CableAnchor` = `"SELECT"`

---
#### Blind type
Spring balance is replaced by crank operated when cable guided.
All motor options are available for both track and cable guided.

##### Blind Type options
Set by `LoadBlindTypeOptions`
if `GuideSystem` = `"CABLE"` then `BlindType` options = `"CRANK OPERATED L/H", "CRANK OPERATED R/H", + all original motor options`
else if `GuideSystem` = `"TRACK"` then `BlindType` options = `"SPRING BALANCED", + all original motor options`
else `BlindType` options = `"'FIXED PANEL', 'FABRIC ONLY'"`

##### Blind Type values - (on `GuideSystem` change only)
Set by `SetBlindTypeValueAndOptions`
if `GuideSystem` changed then `BlindType` = `"SELECT"`

##### Extra affects
Changing the value of `BlindType` (`BlindType_AfterUpdate`) when `GuideSystem` = `"CABLE"` will **reset** the values of the following fields:
- End Plates
- Pelmet Type
- Pelmet Colour
- Bottom Bar Type
- Cable Anchor
- Installation Type
- Bottom Rail Colour

On `BlindType_AfterUpdate` if `GuideSystem` = `"CABLE"` then `CableSpring` options are set based on the value of `BlindType`

---
#### Side Spline Tape
Spline Tape Colour and Spline Tape Length are disabled and set to N/A when cable guided

##### Side Spline Tape Options
Set by `ToggleSideSplineTape`
if `GuideSystem` = `"CABLE"` then `SplineTapeColour` & `SplineTapeLength` disabled + locked
else `SplineTapeColour` & `SplineTapeLength` normal

##### Side Spline Tape values - (on `GuideSystem` change only)
Set by `ToggleAndSetSideSplineValues`
if `GuideSystem` changed and `GuideSystem` = `"CABLE"` then `SplineTapeLength` = `Null` + `SplineTapeColour` = `"N/A"`
else `SkinHeight` + `SplineTapeLength` calculated like normal and `SplineTapeColour` = `"SELECT"`
##### Extra affects
Added `NeedSplineTape` checks to `Fabric_AfterUpdate`, `OverallDropHeight_AfterUpdate` and `BottomBarType_AfterUpdate` as they attempt to calculate `SplineTapeLength`.

`SkinHeight` is set in `ToggleAndSetSideSplineValues`

---
#### End Plates
End plate options are changed to: 135mm Square End Plates, Stainless Steel (No Pelmet), NONE when cable guided.

##### End Plates options
Set by `LoadEndPlateOptions`
if `GuideSystem` = `"CABLE"` then `EndPlates` options = `"135mm Square End Plates", "Stainless Steel (No Pelmet)", "NONE"`
else if `GuideSystem` = `"TRACK"` then `EndPlates` options = previous options
else `EndPlates` options = `"NONE"`

##### End Plates values
Set by `SetEndPlateValueAndOptions`
if `GuideSystem` changed `EndPlates` = `"SELECT"` or `"NONE"`

Set by `BlindType_AfterUpdate`
if `GuideSystem` = `"CABLE"` and `BlindType` = `"FIXED PANEL", "SKIN ONLY", "FABRIC ONLY"` then `EndPlates` = `"NONE"`
else `EndPlates` = `"SELECT"`

##### Extra Affects
Depending on the choice of end plate `EndPlates_AfterUpdate` will set the following fields:
- `PelmetColour`
- `PlasticInsert`
- `PelmetBackBracket`
- `PelmetType` options
- `PelmetLength`
- `PelmetColour` tab stop

---
#### Pelmet Type
Pelmet type options are changed to: 2 Part 135mm SQUARE, NONE, when cable guided. Options are adjusted and the field is auto populated after end plate selection.

##### Pelmet Type options
Set by `LoadPelmetTypeOptions`
if `GuideSystem` = `"CABLE"` then `PelmetType` options = `"2 Part 135mm SQUARE", "NONE"`
else if `GuideSystem` = `"TRACK"` then `PelmetType` options = previous options
else `PelmetType` options = `"NONE"`

##### Pelmet Type values
Set by `SetPelmetTypeValueAndOptions`
if `GuideSystem` changed `PelmetType` = `"SELECT"` or `"NONE"`

Set by `BlindType_AfterUpdate`
if `GuideSystem` = `"CABLE"` and `BlindType` = `"FIXED PANEL", "SKIN ONLY", "FABRIC ONLY"` then `PelmetType` = `"NONE"`
else `PelmetType` = `"SELECT"`

##### Extra affects
On `EndPlates_AfterUpdate`, `PelmetType` options are set based on the value of `EndPlates`
On `EndPlates_AfterUpdate`, `PelmetType` is set based on the value of `EndPlates`

---
#### Springs
An extra input field called Cable Spring was added. 
L/H Spring input field is disabled and Cable Spring enabled  when cable guided
Only allows selection when blind type is crank operated and only allows selection of correct chirality
Auto populates when blind are 2.5m or more wide and disables all other options

##### Cable Spring options
`CableSpring` default options = `"L/H STANDARD", "R/H STANDARD", "NONE"`

Set by `BlindType_AfterUpdate`
If `GuideSystem` = `"CABLE"` and `BlindType` = `"CRANK OPERATED*"` then `CableSpring` options set to match chirality. i.e. if `BlindType` = `CRANK OPERATED L/H` then `CableSpring` options = `"R/H STANDARD", "NONE"`
else `CableSpring` options = `"NONE"`

Set by `TopWidth_AfterUpdate`
If `GuideSystem` = `"CABLE"` and `BlindType` = `"CRANK OPERATED*"` and `TopWidth` >= 2500 then `"NONE"` removed from `CableSpring` options
else if `BlindType` != `"CRANK OPERATED*"` then `CableSpring` options = `"NONE"`

##### Cable Spring values
Set by `SetAndSwapSprings`
If `GuideSystem` != `"CABLE"` then `CableSpring` = `Null`
else `LHSpring` = `Null`

Set by `BlindType_AfterUpdate`
If `GuideSystem` = `"CABLE"` and `BlindType` = `"CRANK OPERATED*"` and `CableSpring` is wrong chirality then `CableSpring` = correct chirality spring
if `GuideSystem` = `"CABLE"` and `BlindType` != `"CRANK OPERATED*"` then `CableSpring` = `"NONE"`

Set by `TopWidth_AfterUpdate`
If `GuideSystem` = `"CABLE"` and `BlindType` = `"CRANK OPERATED*"` and `TopWidth` >= 2500 then `CableSpring` = correct chirality spring
else if `BlindType` != `"CRANK OPERATED*"` then `CableSpring` = `"NONE"`

---
#### Bottom Bar Type
Only Streamline bottom bars are available when cable guided

##### Bottom Bar Type options
Set by `loadBottomBarOptions`
if `GuideSystem` = `"CABLE"` then `BottomBarType` options = `"Streamline", "NONE"`
else `BottomBarType` options = previous options

##### Bottom Bar Type value
Set by `DefaultBottomBarType`
if `GuideSystem` = `"CABLE"` `BottomBarType` = `"Streamline"`
Note: not ideal but works for now I think

Set by `BlindType_AfterUpdate`
if `GuideSystem` = `"CABLE"` and `BlindType` = `"FIXED PANEL", "SKIN ONLY", "FABRIC ONLY"` then `BottomBarType` = `"NONE"`
else `BottomBarType` = `"Streamline"`

---
#### Weight Bars
Weight Bar options are changed to 2x 32x5mm, 4x 32x5mm@1400mm, NONE, when cable guided. Options are adjusted and the field is auto populated after top width input.

##### Weight Bar options
Set by `LoadWeightBarOptions`
if `GuideSystem` = `"CABLE"` then `WeightBars` options = `"2x 32x5mm", "4x 32x5mm@1400mm", "NONE"`
else `WeightBars` options = previous options

Set by `DefaultWeightBars`
If `GuideSystem` = `"CABLE"` and `TopWidth` >= 3000 then `WeightBars` options = `"4x 32x5mm@1400mm", "NONE"`
else if `GuideSystem` = `"CABLE"` and `TopWidth` > 0 then `WeightBars` options = `"2x 32x5mm", "NONE"`
else `LoadWeightBarOptions`
##### Weight Bar value
Set by `DefaultWeightBars`
If `GuideSystem` = `"CABLE"` and `TopWidth` >= 3000 then `WeightBars` = `"4x 32x5mm@1400mm"`
else if `GuideSystem` = `"CABLE"` and `TopWidth` > 0 then `WeightBars` = `"2x 32x5mm"`
else `WeightBars` = `"NONE"`

---
#### Centre Lock
CL Position and CL Flat Bar Colour are disabled and set to N/A when cable guided

##### Centre Lock options
Set by `ToggleCentreLock`
If `GuideSystem` = `"CABLE"` then `CLPosition` and `cbCLFlatBarColour` are locked and disabled
else `CLPosition` and `cbCLFlatBarColour` are unlocked and enabled

##### Centre Lock value
Set by `ToggleAndSetCentreLockValues`
If `GuideSystem` = `"CABLE"` then `CLPosition` and `cbCLFlatBarColour` = `"N/A"`
else `CLPosition` and `cbCLFlatBarColour` = `"SELECT"`

---
#### Helper Spring
Helper spring is disabled and set to `False` when cable guided

---
#### Handle
Handle is disabled and set to `"NONE"` when cable guided

---
### Deductions

---
#### Pelmet Length
`PelmetLength` = `TopWidth` - 7
Set in `EndPlates_AfterUpdate` 
Conditional check in `InstallationType_AfterUpdate`

---
#### Keyway Tube Length
If crank operated `KeyTubeLength` = `TopWidth` - 114
else if using `ALPHA WSER50 30Nm`, `KeyTubeLength` = `TopWidth` - 105
else `KeyTubeLength` = `TopWidth` - 100
Set in `KeyTubeLengthCalc` which is called in lots of places 

---
#### Bottom Bar Length
If `BottomBarType` = `"Streamline"`, `BottomBarLength` = `MinWidth` or `TopWidth` - 118 (Uses `TopWidth` if `MinWidth` isn't set)
else `BottomBarLength` = 0
Set in `BottomBarType_AfterUpdate` and `MinWidth_LostFocus`

---
#### Skin Width
`SkinWidth` = `MinWidth` or `TopWidth` - 86 (Uses `TopWidth` if `MinWidth` isn't set)
Set in `MinWidth_LostFocus`

---
#### Skin Height
`SkinHeight` = `OverallDropHeight` + 120
Set in `ToggleAndSetSideSplineValues`, `Fabric_AfterUpdate`, `OverallDropHeight_AfterUpdate` and `BottomBarType_AfterUpdate`.

---
### Notes
- [x] Need to break load functions up into loading RowSource and modifying set value into separate functions so that it can call the function to load the correct options with out changing the value when changing between records but also allow the value to be changes when other inputs are changed
- [x] Need to change spring inputs to bottom line so the label isn't in the header so when it is changed it isn't confusing. 
- [x] Need to fix and implement pelmet types and colours
- [x] Clarify ALPHA WSER50 30Nm* - 105mm deduction with Nir
- [x] Between records doesn't work for CL stuff - CL needs a rework
- [x] Spline tape length still got calculated - should be fixed
- [x] Check tabbing
- [ ] Functionalise everything??? There is so much repeated code
- [x] install type default to N/A if Skin only etc?
- [x] Need to copy latest revision across
- [x] Springs are cooked
- [ ] left to do
	- [ ] arrange the report
	- [ ] do a fresh set up run
	- [ ] test
	- [ ] quick audit?
- [ ] Cable spring default to N/A for track and NONE for cable
- [ ] Ask about key tube type