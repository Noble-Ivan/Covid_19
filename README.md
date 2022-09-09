# Covid_19
Covid_19  SQL PROJECT
/* Cleaning Data Project */

SELECT *
FROM nashville
ORDER BY propertyaddress
------------------------------------------------------------------------------------------------------------------------

-- Populate the Property Address data
SELECT *
FROM nashville
WHERE propertyaddress is null
/* From research of the dataset, it is obsrved that there are 29 null values in the propertyaddress column. Digging further, I
found out that whenever the Parcelid is the ssame, the propertyaddress is also the same.

So to replace the null values, I would do a SELF-JOIN and thereby get rid of the null values*/
SELECT a.parcelid AS parcelid1, a.propertyaddress , b.parcelid, b.propertyaddress, COALESCE (a.propertyaddress, b.propertyaddress)
FROM nashville a
JOIN nashville b
	ON a.parcelid = b.parcelid
	AND a.uniqueid != b.uniqueid
WHERE a.propertyaddress IS NULL

UPDATE  nashville a
SET propertyaddress = COALESCE (a.propertyaddress, b.propertyaddress)
FROM nashville a
JOIN nashville b
	ON a.parcelid = b.parcelid
	AND a.uniqueid != b.uniqueid
WHERE a.propertyaddress IS NULl

----------------------------------------------------------------------------------------------------------------------------------

-- Breaking out Address into indivdual Columns i.e(Address, City, State)
/* For the property address*/
SELECT propertyaddress
FROM nashville
--WHERE propertyaddress is null
--ORDER BY ParcelID

SELECT 
SUBSTRING(propertyaddress, 1, POSITION(',' IN propertyaddress) -1) AS Address,
SUBSTRING(propertyaddress, POSITION(',' IN propertyaddress) +1, LENGTH(propertyaddress)) AS Address
FROM nashville
 
 -- Now to add the new columns to the table
 
ALTER TABLE Nashville
ADD PropertySplitAddress varchar(100);

UPDATE Nashville
SET PropertySplitAddress = SUBSTRING(propertyaddress, 1, POSITION(',' IN propertyaddress) -1)

ALTER TABLE Nashville
ADD PropertySplitCity varchar(100);

UPDATE Nashville
SET PropertySplitCity = SUBSTRING(propertyaddress, POSITION(',' IN propertyaddress) +1, LENGTH(propertyaddress))


--- For the Owner Address
SELECT 
SPLIT_PART(owneraddress, ',', 1),
SPLIT_PART(owneraddress, ',', 2),
SPLIT_PART(owneraddress, ',', 3)
FROM Nashville

-- Now to add the new columns to the table
 
ALTER TABLE Nashville
ADD OwnerSplitAddress varchar(100);

UPDATE Nashville
SET OwnerSplitAddress = SPLIT_PART(owneraddress, ',', 1)

ALTER TABLE Nashville
ADD OwnerSplitCity varchar(100);

UPDATE Nashville
SET OwnerSplitCity = SPLIT_PART(owneraddress, ',', 2)

ALTER TABLE Nashville
ADD OwnerSplitState varchar(100);

UPDATE Nashville
SET OwnerSplitState = SPLIT_PART(owneraddress, ',', 3)


---------------------------------------------------------------------------------------------------------------------------------
--Change Y and N to 'Yes' and 'NO' in the "Sold as Vacant" column

SELECT DISTINCT (SoldAsVacant), COUNT(SoldAsVacant)
FROM Nashville
GROUP BY SoldAsVacant;

SELECT SoldAsVacant,
CASE WHEN SoldAsVacant = 'N' THEN 'No'
	 WHEN SoldAsVacant = 'Y' THEN 'Yes'
	 ELSE SoldAsVacant
	 END
FROM Nashville

UPDATE Nashville
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'N' THEN 'No'
	 WHEN SoldAsVacant = 'Y' THEN 'Yes'
	 ELSE SoldAsVacant
	 END


--------------------------------------------------------------------------------------------------------------------------
--Removing Duplicates
WITH RowNumCTE AS (
SELECT *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,                             --To find duplicates
				 SaleDate,
				 LegalReference
				 ORDER BY UniqueID
						) row_num

FROM Nashville
--ORDER BY ParcelID
)

DELETE
FROM RowNumCTE                                        --To delete rows in(Code isn't applicable in PostgreSQL)
WHERE row_num > 1
--ORDER BY PropertyAddress


----------------------------------------------------------------------------------------------------------------------------
--DELETE UNUSED COLUMNS
-- Deleting owneraddress columns,propertyaddress and taxdistrict columns since they are not needed.
ALTER TABLE Nashville
DROP COLUMN SaleDate,
DROP COLUMN PropertyAddress,
DROP COLUMN OwnerAddress,
DROP COLUMN TaxDistrict


























