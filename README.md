# Course-Project-I
Course Project I for Cyber Security Base MOOC-course
For this project I used one of this courses task as template. This task being Megaupload. I made it more insecure by disabling password encryption and adding text field that allowed for XSS vulnerability. I also changed the users and their passwords to be more insecure, reducing them to easily guessable and weak. I also identified some of the basic flaws of the code and proposed fixes to them. This report consist of this intro, tools and guides to get the application and vulnerability identification started and list of six vulnerabilities from the OWASP’s list of top 10 vulnerabilities from year 2017 with info how they affect the application, how they can be identified and how they can be fixed.
Tools used:
-	OWASP ZAP 2.7.0, for fuzzing.
-	H-2 Console, for checking passwords in the database.
-	Maven dependency-check version 1.4.4 for identifying vulnerable components.
-	NetBeans with TMC 1.2.1 for writing the code. 
Path to code: https://github.com/RaheemHaslem/Course-Project-I
Guide to start:
1.	Download the ZIP-file
2.	Extract it to place of your own choosing
3.	Open NetBeans
4.	Click “File””Open project”
5.	Select locations where you extracted the file
6.	Choose the file and then the project as active
7.	Run the project
8.	Use sites below to use the application
Sites for the code after running
localhost:8080/login (for logging in)
	Usable usernames/passwords: user1/123456 & user2/12345
localhost:8080/files (for main page)
Issue: A7 XSS vulnerability
Effect: Use of bad input type allows for XSS attacks.
Steps to reproduce:
1.	Log in with credentials (Usernamer: user1, Password: 12345)
2.	In the “Note” input field write: <script>alert("You've been hacked");</script>
3.	Press “Add”
4.	Message window with a text “You’ve been hacked” should appear.
How to fix: Change “th:utext” in files.html to “th:text”.

Issue: A5 Broken access control
Effect: Due to no regulation as to who can access files it is possible for one user to access another’s personal files.
Steps to reproduce
1.	Log in with credentials username: user1, Password: 123456
2.	Press add three times
3.	Type in the URL “localhost:8080/login”
4.	Log in with credentials username: user2, Password: 12345
5.	Type in the URL: localhost:8080/files/3
6.	Browser should now prompt option to download file with id 3, which is another accounts file
How to fix: Change the FileController.java so that it requires only allows download of files owned by account. Done by changing  code beginning from line 37 and ending at line 47 to:
    @RequestMapping(value = "/files/{id}", method = RequestMethod.GET)
    public ResponseEntity<byte[]> viewFile(Authentication authentication, @PathVariable Long id) {
        Account account = accountRepository.findByUsername(authentication.getName());
        FileObject fo = fileRepository.findOne(id);
        if (!account.getFileObjects().contains(fo))
        {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
        final HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.parseMediaType(fo.getContentType()));
        headers.add("Content-Disposition", "attachment; filename=" + fo.getName());
        headers.setContentLength(fo.getContentLength());

        return new ResponseEntity<>(fo.getContent(), headers, HttpStatus.CREATED);
    }

Issue: A2 Broken authentication
Effect: Weak passwords allow for brute forcing with list of weakest passwords. No regulation on how many failed attempts are allowed allows for brute forcing with ease.
Steps to reproduce:
1.	Download list of 500 weakest passwords from https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/500-worst-passwords.txt
2.	Launch the application
3.	Use an application that allows bruteforcing/fuzzing ie. OWASP ZAP
4.	In fuzzing usernames use :user1 and user 2
5.	in fuzzing passwords use the downloaded list
6.	You should be able to identify which passwords both users use and use this information to log in http://localhost:8080/login
How to fix: Set a bad credential event listener to block tries from certain IP address after enough failed attempts (as descripted in https://www.baeldung.com/spring-security-block-brute-force-authentication-attempts), or use passwords that aren’t weak.
Issue: A3 Sensitive data Exposure
Effect: Passwords stored in plain text make it possible to use them as they are if breached.
Steps to replicate
1.	Open files SecurityConfiguration.java and CustomUserDetailsService.java
2.	On the line 33 to 41 of SecurityConfiguration.java you can see that password encoder is left inside comments (Developer probably tested something and forgot this major flaw) as well as in line 21-22 and 28, 33 of CustomUserDetailsService.java.
3.	From this you can conclude that no encryption is in effect.
4.	Type in URL: http://localhost:8080/h2-console/ login window should appear
5.	In the field JDBC URL type: jdbc:h2:mem:testdb
6.	Click “Test Connection” if connection is ok click “Connect”
7.	You should see H2-console, click ACCOUNT a query “SELECT * FROM ACCOUNT” should appear in the SQL statement box.
8.	Click run.
9.	You should see all accounts and their passwords in clear text.
How to fix: Enable the encoders by removing lines above from comments so that passwords are encoded.
Issue: A6 Security misconfiguration
Effect: A disabled security feature allows for CSRF attacks.
Steps to reproduce
1.	Log in with credentials user1, 123456
2.	In the note field type: <form action="<nowiki>http://localhost:8080/files</nowiki>" method="POST">
<input type="text" name="note" value="Test"/>
<input type="submit" value="Add!"/>
</form> 
3.	New line with input field should appear with the text “Test” in it.
4.	This proves that CSRF is possible
5.	Go to SecurityConfiguration.java, you’ll see that security for CSRF is disabled.
How to fix: Remove line 23 from the code so that CSRF tokens are forbidden, or do the fix for issue A7.
Issue: A9 Using components with known vulnerabilities
Effect: Vulnerable components allow for easy exploits as they are known.
Steps to reproduce
1.	Run project
2.	Open command prompt.
3.	Navigate to directory with the pom.xml
4.	Run command: mvn dependency-check:check
5.	Wait for scan to finish.
6.	You should see list of many vulnerable components along with their CVE-number
How to fix: Updating vulnerable components. If a component can’t be updated then check what vulnerability given component allows and change application’s code accordingly to minimize the effects. Possibly checking if all dependencies are necessary and if not remove them.
These were security flaws I managed to create and identify with accuracy. Feel free to identify more if you manage to find them. For security flaws are more likely to be implemented by accident than with purpose and so there might be issues not meant to be.
