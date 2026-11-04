# CodeAlpha_Banking-System-


#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Account {
    int accountNumber;
    char name[50];
    float balance;
};

void createAccount();
void depositMoney();
void withdrawMoney();
void checkBalance();

int main() {
    int choice;
    while (1) {
        printf("\n--- Bank Account Management System ---\n");
        printf("1. Create Account\n");
        printf("2. Deposit Money\n");
        printf("3. Withdraw Money\n");
        printf("4. Balance Enquiry\n");
        printf("5. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: createAccount(); break;
            case 2: depositMoney(); break;
            case 3: withdrawMoney(); break;
            case 4: checkBalance(); break;
            case 5: exit(0);
            default: printf("Invalid choice! Please try again.\n");
        }
    }
    return 0;
}

void createAccount() {
    FILE *fp = fopen("bank_records.dat", "ab");
    if (!fp) {
        printf("Error opening file!\n");
        return;
    }

    struct Account acc;
    printf("Enter Account Number: ");
    scanf("%d", &acc.accountNumber);
    printf("Enter Account Holder Name: ");
    scanf(" %[^\n]%*c", acc.name);
    printf("Enter Initial Deposit Amount: ");
    scanf("%f", &acc.balance);

    if (acc.balance < 0) {
        printf("Initial balance cannot be negative. Account creation failed.\n");
        fclose(fp);
        return;
    }

    fwrite(&acc, sizeof(struct Account), 1, fp);
    fclose(fp);
    printf("Bank account created successfully!\n");
}

void depositMoney() {
    FILE *fp = fopen("bank_records.dat", "rb");
    FILE *temp = fopen("temp.dat", "wb");
    if (!fp || !temp) {
        printf("Error processing file!\n");
        return;
    }

    int accNo, found = 0;
    float amount;
    printf("Enter Account Number: ");
    scanf("%d", &accNo);
    printf("Enter Amount to Deposit: ");
    scanf("%f", &amount);

    if (amount <= 0) {
        printf("Invalid deposit amount!\n");
        fclose(fp);
        fclose(temp);
        remove("temp.dat");
        return;
    }

    struct Account acc;
    while (fread(&acc, sizeof(struct Account), 1, fp)) {
        if (acc.accountNumber == accNo) {
            acc.balance += amount;
            found = 1;
            printf("Successfully deposited %.2f. New Balance: %.2f\n", amount, acc.balance);
        }
        fwrite(&acc, sizeof(struct Account), 1, temp);
    }

    fclose(fp);
    fclose(temp);
    remove("bank_records.dat");
    rename("temp.dat", "bank_records.dat");

    if (!found) {
        printf("Account number %d not found!\n", accNo);
    }
}

void withdrawMoney() {
    FILE *fp = fopen("bank_records.dat", "rb");
    FILE *temp = fopen("temp.dat", "wb");
    if (!fp || !temp) {
        printf("Error processing file!\n");
        return;
    }

    int accNo, found = 0;
    float amount;
    printf("Enter Account Number: ");
    scanf("%d", &accNo);
    printf("Enter Amount to Withdraw: ");
    scanf("%f", &amount);

    if (amount <= 0) {
        printf("Invalid withdrawal amount!\n");
        fclose(fp);
        fclose(temp);
        remove("temp.dat");
        return;
    }

    struct Account acc;
    while (fread(&acc, sizeof(struct Account), 1, fp)) {
        if (acc.accountNumber == accNo) {
            found = 1;
            if (acc.balance >= amount) {
                acc.balance -= amount;
                printf("Successfully withdrew %.2f. Remaining Balance: %.2f\n", amount, acc.balance);
            } else {
                printf("Insufficient balance! Current balance is %.2f\n", acc.balance);
            }
        }
        fwrite(&acc, sizeof(struct Account), 1, temp);
    }

    fclose(fp);
    fclose(temp);
    remove("bank_records.dat");
    rename("temp.dat", "bank_records.dat");

    if (!found) {
        printf("Account number %d not found!\n", accNo);
    }
}

void checkBalance() {
    FILE *fp = fopen("bank_records.dat", "rb");
    if (!fp) {
        printf("No account records found!\n");
        return;
    }

    int accNo, found = 0;
    printf("Enter Account Number: ");
    scanf("%d", &accNo);

    struct Account acc;
    while (fread(&acc, sizeof(struct Account), 1, fp)) {
        if (acc.accountNumber == accNo) {
            printf("\n--- Account Details ---\n");
            printf("Account Number: %d\n", acc.accountNumber);
            printf("Account Holder: %s\n", acc.name);
            printf("Current Balance: %.2f\n", acc.balance);
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("Account number %d not found!\n", accNo);
    }
    fclose(fp);
}
