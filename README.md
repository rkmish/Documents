# Documents

1)update node js to v7:
step 1:curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -

step 2:sudo apt-get install -y nodejs


2)pm2:
sudo ssh -i Downloads/xxx.pem ubuntu@35.163.77.114


3)install pyscharm
step 1:extract downloaded tar.gz file of pycharm communtiy edition 
step 2:/home/rohit/Downloads/pycharm-community-2017.1/bin/pycharm.sh   ---path of extracted pycharm.sh
4)used while loop : for multiple listbox value by comma seperated 
USE [MACWorkflow_DEV]
GO
/****** Object:  StoredProcedure [dbo].[Proc_PU_AddUserFaciltymappingInternational]    Script Date: 07/31/2017 14:37:49 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-- =============================================
-- Author:		<Rohit mIshra>
-- Create date: <26 july 2017>
-- Description:	<Add employee code for multiple facility for particular country >
--exec Proc_PU_AddUserFaciltymappingInternational 'Update','100473',1,'9,10',1,''
-- =============================================
ALTER PROCEDURE [dbo].[Proc_PU_AddUserFaciltymappingInternational] 
(
@Flag varchar(15),
--@Result varchar(15),
@EmployeeId nvarchar(11),
@CountryID int,
@FacilityID nvarchar(50),
@Status bit,
@RecCreatedBy nvarchar(50)
)
AS
BEGIN
	SET NOCOUNT ON;
	
	
	if(@Flag='Insert')
	begin
	Declare @count int,@SplitInsertFlag int 
	set @SplitInsertFlag=1
	Select @count=MAX(id) from [dbo].fnSplit(@FacilityID, ',')
	   while(@count>=@SplitInsertFlag)
	   begin
	   declare @FaclitySplitVAl int
	   set @FaclitySplitVAl = (select list from [dbo].fnSplit(@FacilityID, ',') where id = @SplitInsertFlag)
	   If exists(Select 1 from TD_PU_UserFacilityMapping where EmployeeCode=@EmployeeId and FacilityID=@FaclitySplitVAl and IsActive=1)
				begin
					Select 2 Result
				end
				else
				begin
					Insert into TD_PU_UserFacilityMapping (
							EmployeeCode,
							--CountryID,
							FacilityID,
							IsActive,
							RecCreatedBy,
							RecCreatedDate )
							values (
							@EmployeeID,
							--@CountryID,
							@FaclitySplitVAl,
							@Status,
							@RecCreatedBy,
							getdate())
				Select 1 Result
				end
				set @SplitInsertFlag=@SplitInsertFlag+1
			end
	End
	else if @Flag='Update'
	begin
	Declare @countUpdate int,@SplitUpdateFlag int 
	set @SplitUpdateFlag=1
	Select @countUpdate=MAX(id) from [dbo].fnSplit(@FacilityID, ',')
	
	--select @countUpdate;
	UPDATE TD_PU_UserFacilityMapping set IsActive=0 where EmployeeCode = @EmployeeId
	
	--UPDATE A set A.IsActive=0,RecUpdatedBy=@RecCreatedBy,RecUpdatedDate=getdate() 
	--from TD_PU_UserFacilityMapping A 
	--join TD_PU_CountryFacilityMapping CFM  on CFM.FacilityID = A.FacilityID
	--join TU_User_International UI on UI.CountryID = CFM.CountryID 
	--where UI.EmployeeCode=@EmployeeId and UI.CountryID <> @CountryID 
	while(@countUpdate>=@SplitUpdateFlag)
	--select @SplitUpdateFlag
	begin
	   declare @FaclityUpdateSplitVAl int
	   set @FaclityUpdateSplitVAl = (select list from [dbo].fnSplit(@FacilityID, ',') where id = @SplitUpdateFlag)
	   
		If exists(Select 1 from TD_PU_UserFacilityMapping where EmployeeCode=@EmployeeId and FacilityID=@FaclityUpdateSplitVAl)
					begin
					
						Update TD_PU_UserFacilityMapping Set 
										--FacilityID=@FacilityID,
										IsActive=@Status,
										RecUpdatedBy=@RecCreatedBy,
										RecUpdatedDate=getdate()
							 where employeecode=@EmployeeId 
									and FacilityID=@FaclityUpdateSplitVAl
							Select 1 Result
					end
					else
					begin
					
   						Insert into TD_PU_UserFacilityMapping (
								EmployeeCode,
								--CountryID,
								FacilityID,
								IsActive,
								RecCreatedBy,
								RecCreatedDate )
								values (
								@EmployeeID,
								--@CountryID,
								@FaclityUpdateSplitVAl,
								@Status,
								@RecCreatedBy,
								getdate())
					Select 1 Result
					end
					set @SplitUpdateFlag=@SplitUpdateFlag+1
		end
		
	end	
	else if @Flag='Active_InActive'
	Begin
	
		Update TD_PU_UserFacilityMapping 
		Set IsActive = @Status
		where employeecode=@EmployeeId 

		Select 1 Result
	End			
END

