# share

select * from m_ServiceRate where ID in ('9706c8b6-66e5-4997-9313-70ce497410ad','77bc2182-f267-4536-91ea-ded955d944e0','1f967207-7a43-44ac-a172-acfef129956a','55046469-97e1-403e-a448-3da4599e3663')

CREATE PROCEDURE UpdateServiceRate2
AS
BEGIN
DECLARE @id uniqueidentifier
DECLARE db_cursor CURSOR FOR select ID from m_ServiceRate where SiteID = '7FEF66FB-2962-4A78-8323-119D63D5E6DF' and StartDate = '2018-12-31'
OPEN db_cursor  
FETCH NEXT FROM db_cursor INTO @id  

WHILE @@FETCH_STATUS = 0  
BEGIN  
	Update m_ServiceRate set StartDate = '2018-12-26' where ID = @id

	Insert into m_ServiceRateDaily(ID, ServiceRateID, ApplyDate, IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, CreatedDate, IsDelete)
	select newid(), ServiceRateID , '2018-12-26', IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, GETDATE(), IsDelete from m_ServiceRateDaily where ApplyDate = '2018-12-31' and ServiceRateID = @id

	Insert into m_ServiceRateDaily(ID, ServiceRateID, ApplyDate, IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, CreatedDate, IsDelete)
	select newid(), ServiceRateID , '2018-12-27', IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, GETDATE(), IsDelete from m_ServiceRateDaily where ApplyDate = '2018-12-31' and ServiceRateID = @id

	Insert into m_ServiceRateDaily(ID, ServiceRateID, ApplyDate, IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, CreatedDate, IsDelete)
	select newid(), ServiceRateID , '2018-12-28', IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, GETDATE(), IsDelete from m_ServiceRateDaily where ApplyDate = '2018-12-31' and ServiceRateID = @id

	Insert into m_ServiceRateDaily(ID, ServiceRateID, ApplyDate, IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, CreatedDate, IsDelete)
	select newid(), ServiceRateID , '2018-12-29', IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, GETDATE(), IsDelete from m_ServiceRateDaily where ApplyDate = '2018-12-31' and ServiceRateID = @id

	Insert into m_ServiceRateDaily(ID, ServiceRateID, ApplyDate, IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, CreatedDate, IsDelete)
	select newid(), ServiceRateID , '2018-12-30', IncludeTaxes, NetPrice, RateType, SellPrice, IndexingRequire, StopSell, StopPromotion, Inactive, CreatedBy, GETDATE(), IsDelete from m_ServiceRateDaily where ApplyDate = '2018-12-31' and ServiceRateID = @id


	FETCH NEXT FROM db_cursor INTO @id
END 
CLOSE db_cursor  
DEALLOCATE db_cursor
END
GO
exec dbo.UpdateServiceRate2
GO
drop procedure dbo.UpdateServiceRate2
go
