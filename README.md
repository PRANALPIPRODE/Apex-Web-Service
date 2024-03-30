# Apex-Web-Service
Create an Apex REST class that is accessible at /Accounts/&lt;Account_ID>/contacts. The service will return the account's ID and name plus the ID and name of all contacts associated with the account. Write unit tests that achieve 100% code coverage for the class and run your Apex tests.Create an Apex class

@RestResource(urlMapping='/Accounts/*/contacts')
global with sharing class AccountManager {

    @HttpGet
    global static Account getAccount() {
        RestRequest req = RestContext.request;
        String accountId = req.requestURI.substringBetween('Accounts/', '/contacts');

        Account result = [SELECT Id, Name, (SELECT Id, Name FROM Contacts) FROM Account WHERE Id = :accountId LIMIT 1];
        
        return result;
    }
}

--test
@isTest
private class AccountManagerTest {

    @isTest
    static void testGetAccount() {
        // Create Account
        Account acc = new Account(Name='Test Account');
        insert acc;

        // Create Contacts associated with Account
        Contact con1 = new Contact(AccountId = acc.Id, LastName='Test1');
        Contact con2 = new Contact(AccountId = acc.Id, LastName='Test2');
        insert new List<Contact>{con1, con2};

        // Call the REST service
        Test.startTest();
        RestRequest req = new RestRequest();
        RestResponse res = new RestResponse();
        req.requestURI = '/services/apexrest/Accounts/' + acc.Id + '/contacts';
        req.httpMethod = 'GET';
        RestContext.request = req;
        RestContext.response = res;
        Account result = AccountManager.getAccount();
        Test.stopTest();

        // Assert Account details
        System.assertEquals(acc.Id, result.Id);
        System.assertEquals(acc.Name, result.Name);
        
        // Assert Contact details
        System.assertEquals(2, result.Contacts.size());
        System.assertEquals(con1.Id, result.Contacts[0].Id);
        System.assertEquals(con1.LastName, result.Contacts[0].LastName);
        System.assertEquals(con2.Id, result.Contacts[1].Id);
        System.assertEquals(con2.LastName, result.Contacts[1].LastName);
    }
}


