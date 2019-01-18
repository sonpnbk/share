CREATE TABLE [dbo].[m_SiteGateway] (
    [ID]               UNIQUEIDENTIFIER NOT NULL,
    [SiteId]           UNIQUEIDENTIFIER NULL,
    [PaymentGatewayID] UNIQUEIDENTIFIER NULL,
    [Inactive]         BIT              NOT NULL,
    [IsDelete]         BIT              NOT NULL,
    CONSTRAINT [PK_m_SiteGateway] PRIMARY KEY CLUSTERED ([ID] ASC)
);
ALTER TABLE [dbo].[m_SiteGateway] WITH NOCHECK
    ADD CONSTRAINT [FK_m_SiteGateway_m_Site] FOREIGN KEY ([SiteId]) REFERENCES [dbo].[m_Site] ([ID]);
ALTER TABLE [dbo].[m_SiteGateway] WITH NOCHECK
    ADD CONSTRAINT [FK_m_SiteGateway_m_PaymentGateway] FOREIGN KEY ([PaymentGatewayID]) REFERENCES [dbo].[m_PaymentGateway] ([ID]);
ALTER TABLE [dbo].[m_SiteGateway] WITH CHECK CHECK CONSTRAINT [FK_m_SiteGateway_m_Site];

ALTER TABLE [dbo].[m_SiteGateway] WITH CHECK CHECK CONSTRAINT [FK_m_SiteGateway_m_PaymentGateway];

ALTER TABLE [dbo].[m_PaymentParameter]
  ADD KeyKeyValue nvarchar(250);
INSERT [dbo].[m_PaymentGateway] ([ID], [PaymentCode], [LogoURL], [Inactive], [CreatedBy], [CreatedDate], [UpdatedBy], [UpdatedDate], [IsDelete]) VALUES (N'50253141-c957-4b0b-8b12-aac6e8dd97ed', N'9950', NULL, 0, NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), 0)
INSERT [dbo].[m_PaymentGateway] ([ID], [PaymentCode], [LogoURL], [Inactive], [CreatedBy], [CreatedDate], [UpdatedBy], [UpdatedDate], [IsDelete]) VALUES (N'68e469f4-2400-4b31-b903-b83e95773dfc', N'9954', NULL, 0, NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), 0)
INSERT [dbo].[m_PaymentGateway] ([ID], [PaymentCode], [LogoURL], [Inactive], [CreatedBy], [CreatedDate], [UpdatedBy], [UpdatedDate], [IsDelete]) VALUES (N'e7794f76-598d-4680-b038-d39134375663', N'9954', NULL, NULL, NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), 0)
INSERT [dbo].[m_PaymentGateway] ([ID], [PaymentCode], [LogoURL], [Inactive], [CreatedBy], [CreatedDate], [UpdatedBy], [UpdatedDate], [IsDelete]) VALUES (N'aa8b94a0-b351-4a97-a0d0-d664a851362a', N'9950', NULL, 0, NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), NULL, CAST(N'2018-11-06T04:45:53.083' AS DateTime), 0)
INSERT [dbo].[m_SiteGateway] ([ID], [SiteId], [PaymentGatewayID], [Inactive], [IsDelete]) VALUES (N'c210d0d7-dfed-4dcf-ab9b-1c4aa8e6afac', N'7fef66fb-2962-4a78-8323-119d63d5e6df', N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, 0)
INSERT [dbo].[m_SiteGateway] ([ID], [SiteId], [PaymentGatewayID], [Inactive], [IsDelete]) VALUES (N'358e67ae-3bac-43eb-8992-2d453dfa4160', N'7fef66fb-2962-4a78-8323-119d63d5e6df', N'e7794f76-598d-4680-b038-d39134375663', 0, 0)
INSERT [dbo].[m_SiteGateway] ([ID], [SiteId], [PaymentGatewayID], [Inactive], [IsDelete]) VALUES (N'5da2d809-a820-489e-af3c-6caceb3dab5f', N'93df717a-186c-49ad-b1fb-36d07f3d8681', N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, 0)
INSERT [dbo].[m_SiteGateway] ([ID], [SiteId], [PaymentGatewayID], [Inactive], [IsDelete]) VALUES (N'7d282e55-6bff-4a27-b85c-cdc956dfdc98', N'93df717a-186c-49ad-b1fb-36d07f3d8681', N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, 0)
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'c376e9fc-1519-4544-97e4-02473f1a2983', N'GatewayURL', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'https://onepay.vn/vpcpay/vpcpay.op')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'd109c41c-7f20-4d4b-907a-0caa059c365b', N'AgainLink', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'https://ticket.ttcworld.vn')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'05d4a793-b00b-41a9-a310-11437abecb2f', N'SecretKey', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'159E2FB6135A219257ABF43CD2A6B376')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'183067a3-4f80-4d41-8d4a-1e252843f4b1', N'Version', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'2')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'b1455205-a69e-4134-986d-49dfe08950b0', N'Merchant', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'TTCLAMDONG')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'c2b36d28-d7dc-4eed-95b9-4bb94e20dcef', N'AccessCode', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'07DB6FE6')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'3fea629d-66b0-49c0-bbfe-4ef23dc6a8f9', N'GatewayURL', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'https://onepay.vn/onecomm-pay/vpc.op')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'4022234d-60fe-475d-ba7e-5658cd7dfc4f', N'AccessCode', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'3E8BYTPG')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'8e9c7b5d-82ec-4170-aa15-5efb0a6810bd', N'SecretKey', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'D36446E9BCA1251C5079B3ECBA9CCFBF')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'b05c23ca-e24b-41bd-a274-60e1680b7622', N'Version', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'2')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'8144b456-3b7d-4713-8985-611e9c16dbbb', N'Title', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'TTC WORLD')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'd2158359-ca94-41da-9b4e-62b318af84a9', N'SecretKey', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'A6E3B1F95C514C92A97E048A8431596C')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'9cf6e20e-2a60-4988-93dc-62cb4e0c87df', N'Command', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'pay')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'73512ba6-a4f8-4335-9909-7929dac04650', N'Merchant', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'TTCTACU')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'1d569443-e233-4803-a224-8920f3f6c910', N'Title', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'TTC WORLD')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'be7baeed-d2ef-4512-bd9e-937523575f68', N'AccessCode', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'H7IYTGOB')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'e7ebca49-0f16-4d6a-b862-9d06e90586f9', N'Merchant', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'TTCTACU')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'7df30a7e-c7c8-4cfb-adba-a630c443ad3f', N'Title', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'TTC WORLD')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'9d63ff04-6eb6-4150-9b4e-a7a839b3a797', N'Command', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'pay')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'c69b2257-7196-40d3-bd8e-a81d670e59b9', N'Version', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'2')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'c17cd75e-97a0-4e1d-89d5-b310d35fbd5a', N'Merchant', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'TTCLAMDONG')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'bc1fc657-e496-4612-bc62-b8bb2139db12', N'AgainLink', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'https://ticket.ttcworld.vn')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'74b9f1da-7c77-433c-bbd3-c3dfac126e3b', N'GatewayURL', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'https://onepay.vn/onecomm-pay/vpc.op')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'8b559176-96e2-4fcf-bb91-cdfdc090b18d', N'GatewayURL', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'https://onepay.vn/vpcpay/vpcpay.op')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'0591c06a-0f02-4c53-9a30-d2d2e793952f', N'Title', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'TTC WORLD')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'109d7574-fd3b-4015-bf75-e5aea1b8d56a', N'AccessCode', NULL, N'e7794f76-598d-4680-b038-d39134375663', 0, N'3D9374FB')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'1d054bc2-0827-4e43-955d-e83b133686ce', N'Command', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'pay')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'43feafcb-b7fb-49ee-a4f6-e95ee8d651a7', N'Command', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'pay')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'cab9d7bf-6b77-4c8d-84a5-ea5f8097badb', N'AgainLink', NULL, N'68e469f4-2400-4b31-b903-b83e95773dfc', 0, N'https://ticket.ttcworld.vn')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'dc46d7ed-c772-468c-81f9-f1b128bef9f2', N'SecretKey', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'015AFBD8F29E06C9722C36B7D3A558CC')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'bfb10326-f46a-4231-ba4e-f42263ef98b3', N'AgainLink', NULL, N'50253141-c957-4b0b-8b12-aac6e8dd97ed', 0, N'https://ticket.ttcworld.vn')
INSERT [dbo].[m_PaymentParameter] ([ID], [KeyName], [Description], [PaymentGatewayID], [IsDelete], [KeyValue]) VALUES (N'2bc4f292-442e-4c30-907b-fec491ea6265', N'Version', NULL, N'aa8b94a0-b351-4a97-a0d0-d664a851362a', 0, N'2')
