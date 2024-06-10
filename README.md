# Nashville-Housing-Project

### SQL Code Explanation 💻

#### Step 1: Standardizing Date Format
```sql
Update NashvilleHousing
SET SaleDate = CONVERT(Date,SaleDate)
```
This updates the 'SaleDate' column to ensure all dates are in the same format.

#### Step 2: Populating Property Address Data
```sql
Update a
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
From PortfolioProject.dbo.NashvilleHousing a
JOIN PortfolioProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is null
```
This query fills in missing property addresses by matching records based on the ParcelID.

#### Step 3: Breaking Out Address into Individual Columns
```sql
ALTER TABLE NashvilleHousing
Add PropertySplitAddress Nvarchar(255);

Update NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 )
```
These commands split the 'PropertyAddress' column into separate columns for address, city, and state.

#### Step 4: Changing Y and N to Yes and No in 'Sold as Vacant' Field
```sql
Update NashvilleHousing
SET SoldAsVacant = CASE 
                     When SoldAsVacant = 'Y' THEN 'Yes'
	                 When SoldAsVacant = 'N' THEN 'No'
	                 ELSE SoldAsVacant
                   END
```
This updates 'SoldAsVacant' values to 'Yes' or 'No' for clarity.

#### Step 5: Removing Duplicates
```sql
WITH RowNumCTE AS(
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num
From PortfolioProject.dbo.NashvilleHousing
)
Select *
From RowNumCTE
Where row_num > 1
```
This query identifies and removes duplicate records based on certain criteria.

#### Step 6: Deleting Unused Columns
```sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate
```
Unused columns are dropped from the table.
