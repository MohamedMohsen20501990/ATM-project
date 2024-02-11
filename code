from abc import ABC,abstractmethod
import datetime
import os
from enum import Enum
import uuid
import getpass


#enums enumerations 
class TransactionType(Enum):
    WITHDRAW = "Withdraw"
    DEPOSIT = "Deposit"
    BALANCE_INAUIRY = "Balance Inquiery"
    TRANSFER = "Transfer"

class Bank:
    def __init__(self, name, bank_swift_code):
        self.name = name
        self.bank_swift_code = bank_swift_code
        self.accounts = {}
        
    def add_customer(self, customer):
        for account in customer.accounts.values():
            self.accounts[account.account_number] = account  
                              
class Customer:
    def __init__(self, name, address, phone_number, email):
        self.name = name
        self.address = address
        self.phone_number = phone_number
        self.email = email
        self.accounts = {}
    
    def add_account(self, account):
        self.accounts[account.account_number] = account   

class Account:
    def __init__(self, account_number):
        self.account_number = account_number
        self.balance = 0
        self.linked_card = None
        self.transaction_history = []
    
    def link_card(self,card):
        self.linked_card= card    
    
    def add_transaction(self, transaction):
        self.transaction_history.append(transaction)   
    
    def display_transaction_history(self):
        if not self.transaction_history:
            print("No transactions available")
            return
        print("Printing transaction history: \n")
        for transaction in self.transaction_history:
            print(f"Transaction ID: {transaction.transaction_id}\nTransaction Type: {transaction.transaction_type.value}\nTransaction Amount {transaction.amount}\nTime: {transaction.timestamp}\n")     

class Card:
    def __init__(self, number, pin):
        self.number = number
        self.__pin = pin
        
    # getter for the pin
    def get_pin(self):
        return self.__pin  
    # pin setter
    def set_pin(self, old_pin, new_pin):
        if self.__pin == old_pin:
            self.__pin == new_pin
            return True
        else:
            return False
                         
class Keypad:
    def get_secure_input(self, message):
        return getpass.getpass(message)
        
    def get_input(self, message, secure = False):
        if secure:
            return self.get_secure_input(message)
        return input(message)        
        

            
class Screen:
    def show_message(self, message):
        print(message)
    
    def clear_screen(slef):
        os.system('cls' if os.name == "nt" else 'clear') 

# single responsibility principle        
class CardReader:
    def __init__(self,atm):
        self.atm = atm
        self.authenticator = Authenticator(self.atm.bank)
        
        
    def insert_card(self, card):
        pin = self.atm.keypad.get_input("Please enter yout PIN code: ", secure = True)
        account = self.authenticator.authenticate(card.number, pin)
        if account:
            self.atm.display_main_menu(account)
        else:
            self.atm.screen.show_message("Invalid card or PIN")
            return None     
        
# single responsibility principle    
class Authenticator:
    def __init__(self, bank):
        self.bank = bank
    def authenticate(self, card_number, pin):
        for account in self.bank.accounts.values():
            if account.linked_card and account.linked_card.number == card_number and account.linked_card.get_pin() == pin:
                return account
        return None

class WithdrawHandler:
    def __init__(self, keypad, screen):
        self.keypad = keypad
        self.screen = screen
    
    def handle(self, account):
        while True:
            amount = self.keypad.get_input("Enter the amount to withdraw: ")  
            try:
                amount = float(amount)
                if amount<=0:
                    self.screen.show_message("Invalid amount, please enter a positive value")
                    continue
                transaction = WithdrawTransaction(amount)
                transaction.execute(account)
                break
            except ValueError:
                self.screen.show_message("Invalid input, Please enter a valid amount.")
                
class DepositHandler:
    def __init__(self, keypad, screen):
        self.keypad = keypad
        self.screen = screen
    
    def handle(self, account):
        while True:
            amount = self.keypad.get_input("Enter the amount to Deposit: ")  
            try:
                amount = float(amount)
                if amount<=0:
                    self.screen.show_message("Invalid amount, please enter a positive value")
                    continue
                transaction = DepositTransaction(amount)
                transaction.execute(account)
                break
            except ValueError:
                self.screen.show_message("Invalid input, Please enter a valid amount.")                
                
class BalanceInquiryHandler:

    def handle(self, account):
        transactoin = BalanceInquiry()
        transactoin.execute(account)
        
class TransactionHistoryHandler:
    def handle(self, account):
        account.display_transaction_history()        
                    
class PinChangeHandler:
    def __init__(self, keypad, screen):
        self.keypad = keypad
        self.screen = screen
        
    def handle(self, account):
        old_pin = self.keypad.get_input("Enter your current PIN: ")
        new_pin = self.keypad.get_input("Enter your new PIN: ")
        confirm_new_pin = self.keypad.get_input("Confirm your new PIN: ")
        if new_pin != confirm_new_pin:
            self.screen.show_message("PINs do not match, Please try again")
            return
        if account.linked_card.set_pin(old_pin, new_pin):
            self.screen.show_message("PIN changed successfully!")
        else:
            self.screen.show_message("Incorrect PIN, PIN change failed! ")                                   

class TransferHandler:
    def __init__(self, keypad, screen, bank):
        self.keypad = keypad
        self.screen = screen
        self.bank = bank
    def handle(self, account):
        while True:
            amount = self.keypad.get_input("Enter the amount to transfer: ")
            destination_acount_number = self.keypad.get_input("Enter the destination account number")
            try:
                amount = float(amount)
                if amount<=0:
                    self.screen.show_message("Invalid amount, please enter a positive amount")
                    continue
                transaction= TransferTransaction(amount, destination_acount_number)
                transaction.execute(account, self.bank)
                break
            except ValueError:
                self.screen.show_message("Invalid input, Please enter a valid amount")
                

class AtmInterface:
    def __init__(self, bank, atm_location):
        self.bank = bank
        self.keypad = Keypad()
        self.screen = Screen()
        self.atm_location = atm_location
        self.withdraw_handler = WithdrawHandler(self.keypad, self.screen)
        self.deposit_handler = DepositHandler(self.keypad, self.screen)
        self.balance_inquiry_handler = BalanceInquiryHandler()
        self.transaction_history_handler = TransactionHistoryHandler()
        self.pin_change_handler = PinChangeHandler(self.keypad, self.screen)
        self.transfer_handler = TransferHandler(self.keypad, self.screen, bank)
        self.keypad = Keypad()
        self.screen = Screen()
        
    
        
    def display_main_menu(self, account):
        message = """
        1. Withdraw
        2. Deposit
        3. Balance Inquiry
        4. View Transactions
        5. Change PIN
        6. Transfer Funds
        7. Exit
        Choose an option: """
        while True:
            choice = self.keypad.get_input(message)
            match choice:
                case "1":
                    self.withdraw_handler.handle(account)
                case "2":
                    self.deposit_handler.handle(account)
                case "3":
                    self.balance_inquiry_handler.handle(account)   
                case "4":
                    self.transaction_history_handler.handle(account) 
                case "5":
                    self.pin_change_handler.handle(account)
                case "6":
                    self.transfer_handler.handle(account)    
                case "7":
                    self.screen.show_message("Ejecting card...\nGoodBye!")
                    return
                case _:
                    self.screen.show_message("Invalid choice, please try again")        
            self.keypad.get_input("\n Press Enter to continue")
            self.screen.clear_screen()          
                           
class Transaction(ABC):
    
    
    def __init__(self, transaction_type, amount=None):
        self.transaction_id = uuid.uuid4()
        self.timestamp = datetime.datetime.now()
        self.transaction_type = transaction_type
        self.amount = amount
        
    @abstractmethod
    def execute(self):
        pass     
    
class WithdrawTransaction(Transaction):
    def __init__(self, amount):
        super().__init__(TransactionType.WITHDRAW, amount)
        self.amount = amount
    # polymorphism
    def execute(self, account):
        if account.balance >= self.amount:
            account.balance -= self.amount
            print(f"withdrawl successful, you new balance {account.balance}")
            account.add_transaction(self)
        else:
            print("Insufficent funds")

class DepositTransaction(Transaction):
    def __init__(self, amount):
        super().__init__(TransactionType.DEPOSIT, amount)
        self.amount = amount
        
    def execute(self, account):
        account.balance += self.amount
        print(f"Deposit successful, you new balance {account.balance}")
        account.add_transaction(self)
        
class BalanceInquiry(Transaction):
    def __init__(self):
        super().__init__(TransactionType.BALANCE_INAUIRY, amount=0)
        
    def execute(self,account):
        self.amount=account.balance 
        print(f"your balance is {account.balance}")
        account.add_transaction(self)
       
class TransferTransaction(Transaction):
    def __init__(self, amount, destination_account_number):
        super().__init__(TransactionType.TRANSFER, amount)
        self.destination_account_number = destination_account_number
        
    def execute(self, account, bank):
        if account.balance >= self.amount:
            destination_account = bank.accounts.get(self.destination_account_number)
            if destination_account:
                account.balance -= self.amount
                destination_account.balance += self.amount
                print(f"Transfer successful, Your new balance {account.balance}")
                account.add_transaction(self)
                
            else:
                print("destination account not found")        
        else:
            print("Insufficient funds")
def main():        
    my_bank = Bank("HSBC", "HSBCEGYPTXXX")
    customer_1 =Customer("omar", "42 boulak eldakror", "0121436242", "omar_gmail.com")
    account_1 = Account("12345")
    account_2 = Account("55555")

    customer_1.add_account(account_1) 
    customer_1.add_account(account_2)        
    my_bank.add_customer(customer_1)


    # print(customer_1.accounts)  
            
    card_1 = Card("1255544665888", "1111") 
    account_1.link_card(card_1)
    # print(account_1.linked_card.number)       


    atm = AtmInterface(my_bank, "Elgiza")
    card_reader = CardReader(atm)
    card_reader.insert_card(card_1)

if __name__ == "__main__":
    main()
    
            
        
        
                             
        
                
        
