# Phishing Email Investigation Report

## Findings

**Time:** 2012-01-26 01:41:18 EST

**Sender:** billjobs@microapple.com

**Recipient:** majoronearth@gmail.com

**IOC Domain:** pashter.com

**IOC IP:** 93.99.104.210

**Sending Infrastructure:** emkei.cz

**Suspicious File:** PuzzleToCoCanDa.pdf

**Actual File Type:** ZIP archive (DaughtersCrown.jpeg, GoodJobMajor.pdf, Money.xlsx)

## Investigation

On January 26, 2012, at 01:41:18 EST, TheMajorOnEarth received a suspicious email from an individual claiming to have information about the abducted CoCanDians. The email contained a ransom demand of $1 billion and included an attachment presented as `PuzzleToCoCanDa.pdf`.

Analysis of the email headers revealed an **SPF authentication failure** and a mismatch between the **From** and **Reply-To** addresses, both of which are potential phishing indicators.

The email appeared to originate from `billjobs@microapple.com`, while the Reply-To address was `negeja3921@pashter.com`. The sending infrastructure was also associated with `emkei.cz`, providing additional evidence that the sender identity had been spoofed.

The attachment `PuzzleToCoCanDa.pdf` was then analyzed using HxD and file-signature analysis. Although the filename indicated that the file was a PDF, its first four bytes were `50 4B 03 04`, identifying it as a ZIP archive rather than a PDF.

The archive was extracted using 7-Zip and its contents were examined further. One of the extracted files contained the signature `FF D8 FF E0`, confirming that it was a JPEG image despite its apparent filename. CyberChef was also used to decode Base64-encoded content, while ExifTool and SQRX were used to assist with metadata and unidentified file analysis.

## Who, What, When, Where, Why, How

**Who:** The email was sent using the identity `billjobs@microapple.com`, with replies directed to `negeja3921@pashter.com`. The exact identity of the attacker was not established.

**What:** A suspicious phishing email containing a $1 billion ransom demand and a disguised file attachment was received.

**When:** January 26, 2012, at 01:41:18 EST.

**Where:** The email was delivered to TheMajorOnEarth's email account.

**Why:** The email attempted to use fear, urgency, curiosity, and a ransom demand to pressure the recipient into interacting with the message and its attachment.

**How:** The attacker used a spoofed sender identity and suspicious email infrastructure. The attachment was disguised as a PDF but was actually a ZIP archive containing additional files with different file types.

## Conclusion

The investigation confirmed that the email was a **phishing/social engineering message** and that the attachment used file-type masquerading to conceal its true format.

The SPF authentication failure, From and Reply-To mismatch, suspicious sending infrastructure, and file-signature mismatch provided multiple indicators supporting the finding.

The investigation also demonstrated the importance of validating the actual structure of an attachment rather than relying solely on its filename or extension. HxD and file signature analysis revealed that `PuzzleToCoCanDa.pdf` was actually a ZIP archive.

No evidence from this investigation alone confirmed that the extracted files executed malicious code. Further sandbox or endpoint behavioral analysis would be required to establish whether the files contained or executed malware.

## Recommendations

* Investigate the identified IP address `93.99.104.210` and IOC domain `pashter.com` for related activity.
* Search email logs for other messages associated with `microapple.com`, `pashter.com`, and `emkei.cz`.
* Analyze the extracted files in a controlled sandbox to determine whether they contain or execute malicious code.
* Review endpoint telemetry if the recipient opened or extracted the attachment.
* Investigate any suspicious network connections or processes associated with the extracted files.
* Determine whether other users received similar phishing emails.
* Continue monitoring the identified indicators for related activity.


<img width="1579" height="858" alt="email 1" src="https://github.com/user-attachments/assets/37308ab6-a435-4333-bfea-4caec4024019" />



<img width="800" height="449" alt="Base64 cyberchef" src="https://github.com/user-attachments/assets/32a2f50b-85fd-43e6-83a3-e542422a63d7" />


<img width="519" height="243" alt="pdf" src="https://github.com/user-attachments/assets/95b331c1-4754-4e29-b1df-d420c1405225" />



<img width="722" height="425" alt="Hiddenfiles" src="https://github.com/user-attachments/assets/8a8a867f-f66d-4417-8be2-1f24a828308c" />



<img width="704" height="420" alt="bound 2 indicate picture jpg" src="https://github.com/user-attachments/assets/20cba2f6-e895-4742-bf55-dc0b66c4e2ef" />
 
