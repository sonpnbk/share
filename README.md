INSERT [dbo].[m_Member] 
([ID], [Fullname], [Gender], [Address], [BirthDay], [Email], [AlternativeEmail], [CellPhone], [CardIssueCount], [PhoneNumber], [PostalCode], [WardID], [NationalityID], [IdentityCard], [MemberType], [ProfileID], [Username], [Password], [Avatar], [Inactive], [CreatedBy], [CreatedDate], [UpdatedBy], [UpdatedDate], [IsDelete]) 
VALUES 
(N'2414e81b-fbc5-441f-a95f-342e8a10ee92', N'Lưu Đại Lương', N'Male', N'Ha Noi, Vietnam', CAST(N'2018-11-02' AS Date), N'', NULL, N'0987654321', 0, N'0987654321', N'110000', N'545eaec9-f87c-4ea1-9e5e-34ffd7ec7903', NULL, N'123456789', N'TRAVELLER', N'd171f03f-4f8f-45e7-9890-7ef649ff9f9e', N'traveller', N'ANrvxrcjALBIS4uGuqWCvEQnLuW0VchVhy9auv4uEhyn8LKLSgt9Gzb+z0ncz2Frjg==', 0x, 0, N'luongld', CAST(N'2018-11-02T17:00:00.000' AS DateTime), NULL, NULL, 0)
GO
ALTER PROCEDURE [dbo].[SP_GetTransaction]
	@transactionDate datetime = '2019-03-11'
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

  SELECT a.*,(a.AmountGross + a.Vat + a.ServiceCharge) AS TotalMoney 
	FROM (
	SELECT	i.SiteCode,i.InvoiceCode,oi.InvoiceCode AS OriginInvoiceCode,i.InvoiceDate,s_i18n.Title AS 'ServiceName',p.GihotechCode AS 'Gihotech_TACode',
			s.GihotechCode AS 'Gihotech_ServiceCode','VÉ' AS Unit,bd.Quantity,
			(
				SELECT		IIF(SUM(bd_vat.Amount) IS NULL, 0, ABS(SUM(bd_vat.Amount)))
				FROM		dbo.BookingDetail bd_vat JOIN dbo.m_Service s_vat ON s_vat.ID = bd_vat.ServiceID
				WHERE		bd_vat.BookingID = bd.BookingID AND bd_vat.ServicePackageID = bd.ServicePackageID AND 
							bd_vat.RowState = 2 AND bd_vat.GroupBy = bd.GroupBy AND 
							bd_vat.IsSplit = 0 AND s_vat.ServiceType = 'VAT' AND s_vat.ServiceFilter = 'VAT' 
			) AS Vat,
			(
				SELECT		IIF(SUM(bd_sc.Amount) IS NULL, 0, ABS(SUM(bd_sc.Amount)))
				FROM		dbo.BookingDetail bd_sc JOIN dbo.m_Service s_sc ON s_sc.ID = bd_sc.ServiceID
				WHERE		bd_sc.BookingID = bd.BookingID AND bd_sc.ServicePackageID = bd.ServicePackageID AND 
							bd_sc.RowState = 2 AND bd_sc.GroupBy = bd.GroupBy AND 
							bd_sc.IsSplit = 0 AND s_sc.ServiceFilter = 'SERVICE_CHARGE' 
							
			) AS ServiceCharge,
			(
				SELECT		IIF(SUM(bd_ag.Amount) IS NULL, 0, ABS(SUM(bd_ag.Amount)))
				FROM		dbo.BookingDetail bd_ag JOIN dbo.m_Service s_ag ON s_ag.ID = bd_ag.ServiceID
				WHERE		bd_ag.BookingID = bd.BookingID AND bd_ag.ServicePackageID = bd.ServicePackageID AND 
							bd_ag.GroupBy = bd.GroupBy AND bd_ag.IsSplit = 0 AND bd_ag.RowState IN (0,1) AND
							bd_ag.ServiceID = bd.ServiceID AND bd_ag.InvoiceID = bd.InvoiceID
			) AS AmountGross,
			(
				SELECT		IIF(SUM(bd_d.Amount) IS NULL, 0,ABS(SUM(bd_d.Amount)))
				FROM		dbo.BookingDetail bd_d JOIN dbo.m_Service s_ag ON s_ag.ID = bd_d.ServiceID
				WHERE		bd_d.BookingID = bd.BookingID AND bd_d.ServicePackageID = bd.ServicePackageID AND 
							bd_d.GroupBy = bd.GroupBy AND bd_d.IsSplit IN (0,1) AND bd_d.RowState = 0 AND
							bd_d.PromotionID IS NOT NULL AND bd_d.Amount < 0 AND 
							bd_d.ServiceID = bd.ServiceID AND bd_d.InvoiceID = bd.InvoiceID
			) AS Discount


	FROM	dbo.Invoice i WITH(NOLOCK) JOIN 
			dbo.BookingDetail bd WITH(NOLOCK) ON i.ID = bd.InvoiceID JOIN
			dbo.Booking b WITH(NOLOCK) ON b.ID =  bd.BookingID JOIN
			dbo.m_Service s WITH(NOLOCK) ON s.ID = bd.ServiceID JOIN
			dbo.m_Service_i18n s_i18n WITH(NOLOCK) ON s_i18n.ServiceID = bd.ServiceID AND s_i18n.LangCode = 'vi' LEFT JOIN 
			dbo.m_Profile p WITH(NOLOCK) ON b.ProfileID = p.ID AND p.ProfileCode <> 'default_pos' LEFT JOIN  
			dbo.Invoice oi WITH(NOLOCK) ON oi.ID = bd.OriginInvoiceID  

	WHERE DATEDIFF(DAY,i.InvoiceDate,@transactionDate) = 0 AND  bd.IsSplit IN (1,0) AND bd.RowState = 0
	AND bd.PromotionID IS NULL 
	AND i.Inactive = 0
	) a
	ORDER BY a.SiteCode,a.InvoiceDate DESC
END
