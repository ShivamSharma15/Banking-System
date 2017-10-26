import random
import cx_Oracle
from collections import Counter
from datetime import datetime
import sys
import calendar

# below is the class for admin sign in which all the related work to admin is performed
class AdminSignIn:

    def __init__(self):
        print(" do you wish to see the closed Account histories")
        answer= raw_input(" enter Y for YES and N for NO and to be logged out")
        if( answer== 'Y'):
            con= cx_Oracle.connect(" user1/user123")
            cur= con.cursor()
            cur.execute(" select * from CustomerAccountDetails where Validity= 'CLOSED'")
            print( cur.fetchall())
        else:
            print(" you are being logged out")
            return

# the below class is for the Sign Up for a new Customer
        

class SignUp:

    def sending_to_database(self):
        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " INSERT INTO CustomerAccountDetails values( :param1, :param2, :param3, :param4, :param5 ,:param6, :param7, :param8, :param9, :param10)", { 'param1': self.accountno, 'param2': self.fname, 'param3': self.lname, 'param4': self.addressline1, 'param5': self.addressline2, 'param6':self.city, 'param7':self.pin, 'param8':self.password, 'param9': self.status, 'param10':'OPEN'})
        cur= con.cursor()
        cur.execute( " select * from CustomerAccountDetails")
        print( cur.fetchall())
        if( self.accounttype=='Savings'):
            cur.execute(" INSERT INTO SavingAccountCustomers(AccountNo, InterestRate, Withdrawls, Balance) values( :param1, 7.5,10, :param2)", {'param1':self.accountno, 'param2': self.balance})
        elif( self.accounttype == 'Current'):
            cur.execute( " Insert into CurrentAccountCustomers values( :param1, :param2)", {'param1':self.accountno, 'param2':self.balance})
        cur.execute( " select * from SavingAccountCustomers")
        print( cur.fetchall())
        cur.execute( " select * from CurrentAccountCustomers")
        print( cur.fetchall())
        con.commit()
        con.close()
        

    def __str__(self):
        return ( self.name + str( self.balance))

   


        
    def __init__(self):
         self.customerid= random.randrange(100000,9999999999, 1)
         self.accountno=  "BANK"+ str( self.customerid)
         self.status=  'UNLOCKED'
         self.fname= raw_input("first name")
         self.fname= (self.fname.lower())
         self.lname= raw_input("last name")
         self.lname= (self.lname.lower())
         self.name= self.fname + self.lname
         self.addressline1= raw_input(" address Line 1")
         self.addressline2= raw_input(" Address Line 2")
         self.city= raw_input("city")
         pinflag= False

         # the checking off pin is performed
         while(pinflag== False):
             try:
                 pint= (raw_input(" Pin( 6 digits)"))
                 pcounter=0
                 for i in pint:
                     pcounter += 1
                 pint= int(pint)
                 if( pcounter == 6):
                     self.pin= pint
                     pinflag= True
                 else:
                     raise Exception("no pin is greater than 6 digits")
             except  Exception as err:
                print('caught this error', repr(err))
         self.address= self.addressline1 +  " "+self.addressline2+  " " + self.city +  " "+ str( self.pin)
         accc= int(raw_input(" enter the account type you wish for , enter 1 for Savings and 2 for Current"))
         if( accc== 1):
             self.accounttype= "Savings"
             self.balance= 0
             
             
             
         elif( accc== 2):
             self.accounttype= "Current"
             self.balance= 5000
             
             
         #checking the password
         passflag= False
         while( passflag== False):
             try:
                 passc= raw_input("password( 8 alphanumeric to be strong)")
                 if(len(passc) < 8):
                     raise Exception('the password must be 8 characters long')
                 try:
                     passc= int(passc)
                     print("password requires atleast 1 alphahabet to be strong and valid")
                 except Exception as err:
                     #print(' exception is thrown', repr(err))
                     print("password accepted !!! remember it")
                     passflag= True
                     self.password= passc
                 #if( isinstance(passc, int)== True ):
                 #   raise Exception("password requires atleast 1 alphahabet to be strong and valid")
             except Exception as err:
                 print('caught this error', repr(err))
                 passflag= False
                 
        
        

# the class is for Sign In it works with the drop down menu for Sign In

class SignIn:

    def account_closure( self, accountnou):
        print(    "BANK SERVICES ")
        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " update CustomerAccountDetails set Validity= 'CLOSED' where AccountNo= :param1", {'param1':accountnou})
        print(" YOUR ACCOUNT HAS BEEN CLOSED")
        con.commit()
        
    def logout(self):
        print("you are logged out")
        return


    def transfer_money( self, accountnofrom):
        accountnoto= raw_input(" enter the accountno in which you want the money to be transfreered")
        con=cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        try:
            cur.execute( " select count(*) from CustomerAccountDetails where AccountNo= :param1", {'param1':accountnoto})
            data= cur.fetchall()
            if( int( data[0][0])== 0):
                raise Exception(" THE ACCOUNT IN WHICH MONEY HAS TO BE TRANSFERRED IS INVALID")

            amount= int( raw_input( " enter the amount you want to transfer"))
            if( amount < 0):
                raise Exception(" THE AMOUNT ENTERED IS INVALID")

            cur.execute( " select AccountType from CustomerAccountDetails where AccountNo= :param1", { 'param1':accountnofrom})
            data1= cur.fetchall()
            cur.execute( " select AccountType from CustomerAccountDetails where AccountNo= :param2", { 'param2': accountnoto})
            data2= cur.fetchall()

            if( str( data1[0][0])== 'Savings'):
                cur.execute(" select Balance from SavingAccountCustomers where AccountNo= :param1",{'param1':accountnofrom})
                data= cur.fetchall()
                previousbalance= int( data[0][0])
                if( amount> previousbalance):
                    raise Exception( "TRANSFER AMOUNT IS GREATER THAN AVAILABLE BALANCE")
                finalbalance= previousbalance- amount
                cur.execute( " update SavingAccountCustomers  set Balance= :param1 where AccountNo= :param2", {'param1':finalbalance,'param2':accountnofrom})
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Transfer',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':finalbalance, 'param3':accountnofrom})
            else:
                cur.execute(" select Balance from CurrentAccountCustomers where AccountNo= :param1",{'param1':accountnofrom})
                data= cur.fetchall()
                previousbalance= int( data[0][0])
                previousbalance -= 5000
                if( amount> previousbalance):
                    raise Exception( "TRANSFER AMOUNT IS GREATER THAN AVAILABLE BALANCE(we are maintaining min 5k balance)")
                finalbalance= previousbalance- amount
                finalbalance += 5000
                cur.execute( " update CurrentAccountCustomers  set Balance= :param1 where AccountNo= :param2", {'param1':finalbalance,'param2':accountnofrom})
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Transfer',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':finalbalance, 'param3':accountnofrom})

            if( str( data2[0][0]) == 'Savings'):
                cur.execute( " select Balance from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnoto})
                data= cur.fetchall()
                previousbalance= int( data[0][0])
                finalbalance= previousbalance + amount
                cur.execute( " update SavingAccountCustomers set Balance= :param1 where AccountNo= :param2", {'param1':finalbalance ,'param2':accountnoto})
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Transfer',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':finalbalance, 'param3':accountnoto})
                raise Exception("SUCCESS")
                
            else:
                cur.execute( " select Balance from CurrentAccountCustomers where AccountNo= :param1", {'param1':accountnoto})
                data= cur.fetchall()
                previousbalance= int( data[0][0])
                finalbalance= previousbalance + amount
                cur.execute( " update CurrentAccountCustomers set Balance= :param1 where AccountNo= :param2", {'param1':finalbalance ,'param2':accountnoto})
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Transfer',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':finalbalance, 'param3':accountnoto})
                raise Exception("SUCCESS")
            
        except Exception as err:
            con.commit()
            print('Caught this error', repr(err))
        

    def print_statement(self ,accountnou):
        print(" enter the dates from where to where you want to print your account statement in the ")
        dateflag= False
        while( dateflag== False):
            date1= raw_input(" enter the FROM date in the 'YYYY-MM-DD' format")
            year1,month1, day1 = (date1.split('-'))
            date2= raw_input(" enter the  TO date in the 'YYYY-MM-DD' format")
            year2,month2, day2 = (date2.split('-'))
            if( year1< year2):
                dateflag= True
            elif( year1 == year2 and month1< month2):
                dateflag= True
            elif( year1== year2 and month1== month2 and date1 < date2):
                dateflag= True
            else:
                dateflag= False


            con= cx_Oracle.connect("user1/user123")
            cur= con.cursor()
            print(" this is your account statement within the time mentioned above") 
            cur.execute( " select * from CustomerTransactions where AccountNo= :param1 and DateofTransaction >=:param2 and DateofTransaction <= :param3",{ 'param1':accountnou, 'param2': datetime(int(year1), int(month1),int(day1)) , 'param3':datetime(int(year2), int(month2),int(day2))})
            print(cur.fetchall())

    
            

    def withdraw_money( self, accountnou):
        #accountnou= raw_input(" enter the account number from which you want to withdraw the money")
        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " select count(*) from CustomerAccountDetails where AccountNo= :param1", {'param1':accountnou})
        data= cur.fetchall()
        try:
            if( data[0][0]==0):
                raise Exception(" ACCOUNT DOESN'T EXSISTS , ENTER A VALID ACCOUNT NO")
            
            cur.execute( " select count(*) from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
            data= cur.fetchall()
            if( data[0][0]== 1):
                cur.execute( " select Balance from SavingAccountCustomers where AccountNo= :param1", { 'param1':accountnou})
                data= cur.fetchall()
                amount= int( raw_input(" enter the amount to be withdrwan"))
                if( amount< 0):
                    raise Exception(" the amount is illogical , you cant withdrwa nothing")
                if( int(data[0][0]) < amount):
                    raise Exception( " THE AVAILABLE BALANCE IS LESS THAN THAT WISHED TO BE WITHDRWAN")
                cur.execute(" select Lastdate from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
                data= cur.fetchall()
                if( str(data[0][0])== 'None'):
                    cur.execute( " select Balance from SavingAccountCustomers where AccountNo= :param1", { 'param1':accountnou})
                    data= cur.fetchall()
                    previousbalance= int( data[0][0])
                    newbalance= previousbalance - amount
                    print("ch1")
                    #cur.execute( " insert into CustomerTransactions values( auto_increment.nextval, 'Withdrawl',:param1, :param2,:param3, sysdate",{ 'param1':amount, 'param2':newbalance,'param3':accountnou})
                    cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Withdrawl',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':newbalance, 'param3':accountnou})
                    cur.execute( " update SavingAccountCustomers set Balance= :param1, LastDate= sysdate where AccountNo= :param2",{ 'param1':newbalance, 'param2':accountnou})
                    print("The updated details below")
                    cur.execute(" select * from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
                    print( cur.fetchall())
                    cur.execute( " select * from CustomerTransactions where AccountNo= :param1", {'param1': accountnou})
                    print( cur.fetchall())
                    con.commit()
                    raise Exception(" SUCESSS")

                cur.execute( " select extract( year from LastDate ), extract( month from LastDate) from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
                data= cur.fetchall()
                currentyear= datetime.now().year
                currentmonth= datetime.now().month
                if( int( data[0][0]) < currentyear):
                    if( int( data[0][1]) < currentmonth):
                        cur.execute( " select Withdrawls from SavingAccountCustomers where AccountNo= :param1", {'param1': accountnou})
                        data= cur.fetchall()
                        if( int( data[0][0]) != 0):
                            cur.execute( " select Balance from SavingAccountCustomers where AccountNo= :param1", { 'param1':accountnou})
                            data= cur.fetchall()
                            previousbalance= int( data[0][0])
                            newbalance= previousbalance - amount
                            cur.execute( " insert into CustomerTransactions values( auto_increment.nextval, 'Withdrawl',:param1, :param2,:param3, sysdate) ",{ 'param1':amount, 'param2':newbalance,'param3':accountnou})
                            cur.execute( " update SavingAccountCustomers set Balance= :param1, LastDate= sysdate where AccountNo= :param2",{ 'param1':newbalance, 'param2':accountnou})
                            print("The updated details below")
                            cur.execute(" select * from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
                            print( cur.fetchall())
                            cur.execute( " select * from CustomerTransactions where AccountNo= :param1", {'param1': accountnou})
                            print( cur.fetchall())
                            con.commit()
                            raise Exception(" SUCESSS")
                            
                        else:
                            raise Exception( " yuo have passed your withdrwal limit")
                    
            cur.execute( " select count(*) from CurrentAccountCustomers where AccountNo= :param1", {'param1':accountnou})
            data= cur.fetchall()
            if( data[0][0] == 1):
                amount= int( raw_input( " enter the amount you wish to withdraw"))
                if( amount< 0):
                    raise Exception( " enter a valid amount ")
                cur.execute( " select Balance from CurrentAccountCustomers where AccountNo= :param1", {'param1': accountnou})
                data= cur.fetchall()
                allowedbalance= int( data[0][0]) -5000
                if( amount> allowedbalance):
                    raise Exception(" you don't have enough cash fro withdrawl while maintaining min 5K balance")
                newbalance= allowedbalance- amount
                newbalance += 5000
                cur.execute( " update CurrentAccountCustomers set Balance= :param1 where AccountNo= :param2", {'param1': newbalance, 'param2':accountnou})
                cur.execute( " insert into CustomerTransactions values( auto_increment.nextval, 'Withdrawl', :param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':newbalance, 'param3':accountnou})
                print("The updated details below")
                cur.execute(" select * from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
                print( cur.fetchall())
                cur.execute( " select * from CustomerTransactions where AccountNo= :param1", {'param1': accountnou})
                print( cur.fetchall())
                con.commit()
                raise Exception("Sucess")
            
        except Exception as err:
            print('caught this message', repr(err))
            return
                        

    def password_check( self, passwordc, passwordcheck):
        return ( Counter(passwordc)== Counter(passwordcheck))
    

    def deposit_money(self):
        accountnou= raw_input(" enter the account number in which you want to deposit the money")
        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " select count(*) from CustomerAccountDetails where AccountNo= :param1", {'param1':accountnou})
        data= cur.fetchall()
        try:
            if( data[0][0]==0):
                raise Exception(" ACCOUNT DOESN'T EXSISTS , ENTER A VALID ACCOUNT NO")

            amount= int(raw_input(" enter the amount to be deposited"))
            if( amount <=0):
                raise Exception( "ENTER VALID AMOUNT FOR DEPOSITION ")

            cur.execute( " select count(*) from SavingAccountCustomers where AccountNo= :param1", {'param1':accountnou})
            data= cur.fetchall()
            if( data[0][0] == 1):
                cur.execute( "select Balance from SavingAccountcustomers where AccountNo= :param1", {'param1':accountnou})
                data= cur.fetchall()
                print(" previous balance in acount no", accountnou, "is", data[0][0])
                previousbalance= int(data[0][0])
                newbalance= previousbalance + amount
                cur.execute( " update SavingAccountCustomers set Balance= :param1 where AccountNo= :param2", { 'param1':newbalance, 'param2':accountnou})
                print("updated details below")
                cur.execute( "select * from SavingAccountcustomers where AccountNo= :param1", {'param1':accountnou})
                print(cur.fetchall())
                print("recording this transaction in transaction history")
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Deposit',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':newbalance, 'param3':accountnou})
                con.commit()

            cur.execute( " select count(*) from CurrentAccountCustomers where AccountNo= :param1", {'param1':accountnou})
            data= cur.fetchall()
            if( data[0][0] == 1):
                cur.execute( "select Balance from CurrentAccountcustomers where AccountNo= :param1", {'param1':accountnou})
                data= cur.fetchall()
                print(" previous balance in acount no", accountnou, "is", data[0][0])
                previousbalance= int(data[0][0])
                newbalance= prevoiusbalance + amount
                cur.execute( " update CurrentAccountCustomers set Balance= :param1 where AccountNo= :param2", { 'param1':newbalance, 'param2':accountnou})
                print("updated details below")
                cur.execute( "select * from CurrentAccountcustomers where AccountNo= :param1", {'param1':accountnou})
                print(cur.fetchall())
                print("recording this transaction in transaction history")
                cur.execute("insert into CustomerTransactions values(auto_increment.nextval, 'Deposit',:param1, :param2, :param3, sysdate)", {'param1':amount, 'param2':newbalance, 'param3':accountnou})
                con.commit()
        except Exception as err:
            print('caught this error', repr(err))


             

    def address_change( self, accountno):
        newaddressline1= raw_input(" enter the line 1 of the new address")
        newaddressline2= raw_input(" enter the line2 of the new address")
        newcity= raw_input(" enter new city name")
        newpin= raw_input(" enter the new PIN")
        try:
            con= cx_Oracle.connect("user1/user123")
            cur= con.cursor()
            cur.execute(" update CustomerAccountDetails  Set AddressLine1= :param2 , AddressLine2= :param3 , City= :param4 , Pin= :param5 where AccountNo= :param1", { 'param1':accountno, 'param2': newaddressline1, 'param3': newaddressline2, 'param4': newcity, 'param5':newpin})
            cur.execute( " select * from CustomerAccountDetails where AccountNo= :param1", {'param1':accountno})
            print(cur.fetchall())
            print("SUCESSS")
            con.commit()
        except Exception as err:
            print('caught this error', repr(err))
        



      

    def __init__(self):
        print("  BANK SERVICES, inorder to sign in")
        accountnoc= raw_input("enter the account number")
        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " select Password,Status from CustomerAccountDetails where AccountNo= :param1", { 'param1':accountnoc})
        data= cur.fetchall()
        passwordcheck= str(data[0][0])
        statuscheck= str( data[0][1])
        
        try:
            passflag= False
            trials =0
            while( passflag == False and trials !=3):
                passwordc= raw_input(" enter the password")
                if( statuscheck == 'LOCKED'):
                    raise Exception(" The Account is locked contact adminstrator for further process")
                passflag=self.password_check( passwordc, passwordcheck)
                if(passflag == False):
                    trials += 1

                
                if( trials==3):
                    con= cx_Oracle.connect("user1/user123")
                    cur= con.cursor()
                    cur.execute(" update CustomerAccountDetails set Status= 'LOCKED' where AccountNo= :param1", {'param1': accountnoc})
                    cur.execute(" select * from CustomerAccountDetails where AccountNo= :param1",{ 'param1' : accountnoc})
                    print( cur.fetchall())
                    
                    raise Exception(" 3 consecutive retrials are over contact adminstrator for the account is now locked")
        except Exception as err:
            print('Caught this error', repr(err))
            return
                
                    
            
            


        

        con= cx_Oracle.connect("user1/user123")
        cur= con.cursor()
        cur.execute( " select Fname, Lname from CustomerAccountDetails where AccountNo= :param1", { 'param1': accountnoc})
        data= cur.fetchall()
        print(" welcome ", data[0][1], data[0][0])
        print(" select what service you are here to avail based on the serial no")
        print("1. Address Change")
        print("2. Money Deposit")
        print("3. Money withdrwal")
        print("4. Print Statement")
        print("5. Transfer Money")
        print("6. Account Closure")
        print("7. Customer LogOut")
        choice= int(raw_input( " select the service you want to avail according to the serial number"))

        if( choice== 1):
            self.address_change(accountnoc)
        elif( choice== 2):
            self.deposit_money()
        elif(choice==3):
            self.withdraw_money(accountnoc)
        elif(choice==4):
            self.print_statement(accountnoc)
        elif(choice==5):
            self.transfer_money( accountnoc)
        elif( choice==6):
            self.account_closure( accountnoc)
        elif( choice== 7):
            self.logout()
            return

        

        
                



print("  BANK SERVICES")
print("MAIN MENU")
print(" Enter the choices")
print( "1. Sign Up (New Customer)")
print( "2. Sign In (Exsisting Customer)")
print ( "3. Admin Sign In ")
print( "4. Quit")
choice= int(raw_input("Enter the choices ."))

if(choice==1):
    #creating the child of a SignUp class
    print(" welcome to the BANK SERVICES provide the details ")
    SignUpObj= SignUp()
    print(SignUpObj)
    #now we are sending the values to the database in multiple tables
    SignUpObj.sending_to_database()
    
    
    
elif( choice== 2):
    # creating the child of the SignIn class
    SignInObj= SignIn()
    
elif( choice==3):
    # letting signup as the admin
    print(" this is the Admin login, there's only 1 adminstrator")
    adminid= raw_input(" enter the admin id")
    adminpass= raw_input(" enter the admin password")
    con= cx_Oracle.connect("user1/user123")
    cur= con.cursor()
    cur.execute( " select adminpass from AdminAccounts where adminid= :param1", {'param1':adminid})
    data= cur.fetchall()
    if( str( data[0][0]) != adminpass):
        print("ADMINSTRATOR LOGIN NOT ALLOWED")
    else:
        AdminSignInObj= AdminSignIn()
elif( choice ==4):
    try:
        #terminate the whole script"
        print("THANK YOU")
        sys.exit()
    except:
        pass
