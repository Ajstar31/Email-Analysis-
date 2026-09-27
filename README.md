
#Investigation Complete: Email Header & Phishing Attachment Analysis (CoCanDa Lab)

Platform: BlueTeamLabs Online Category: Email Analysis / Digital Forensics Skills: Email header forensics, SPF/DKIM/DMARC validation, file signature (magic byte) analysis, hex analysis, OSINT

 Scenario:
CoCanDa, a planet known as "The Heaven of the Universe," has been suffering a wave of unrest after a mysterious force began abducting its citizens. Following a high-level war-room meeting, the Planetary President learned his own daughter had also disappeared. Days later, with no ransom demand and no leads, a CoCanDa Army Major stationed on Earth received a suspicious email — the first real clue in the case.
As the analyst, my task was to determine whether the email was legitimate, trace its true origin, and safely examine the attached file to uncover what it actually contained.

 Tools Used:
Notepad++
HxD (Hex editor — inspecting raw file bytes / magic numbers)
CyberChef (Base64 decoding and Hex conversion)
Gary Kessler File Signature Table (Reference for identifying true file types from magic bytes)
SQRX (sqrx.com)- (Online multi-format file viewer for unidentified/renamed files)
ExifTool (Metadata extraction)


 Investigation Methodology:
My approach to this email analysis followed three stages:
1.Content review – assess the tone/language of the email body for social‑engineering indicators (urgency, fear, financial threat, curiosity).
2. Authentication analysis – validate SPF, DKIM, and DMARC results, and review the full header chain.
3. Header deep-dive – manually trace Received, Return-Path, From, Reply-To, sending domain, and source IP to establish the email's true origin.

1. Email Header Analysis:
Breaking down the raw header, field by field:
Delivered-To — the actual mailbox that received the message: majoronearth@gmail.com.
Received — the chain of mail servers the message passed through. Reading a Received chain works bottom-up: the entry closest to the bottom of the header block is the server closest to the sender, while the entry at the top is the server closest to the recipient.
X-Google-Smtp-Source — an optional header added by Google's mail infrastructure (or other tooling) during transit; not present on every email.
ARC-Seal (Authenticated Received Chain) — a set of headers used to cryptographically verify that trusted intermediate mail servers handled and did not tamper with the message.
Return-Path — billjobs@microapple.com. This is the address that bounce/failure notifications are sent to, and it's often used for delivery troubleshooting. It does not have to match the visible From address, which is itself already a red flag when combined with other findings.
Received-SPF — result: fail. The record read: "domain of billjobs@microapple.com does not designate 93.99.104.210 as permitted sender." In plain terms, the domain microapple.com never authorized the IP 93.99.104.210 to send mail on its behalf — strong evidence of spoofing.
From vs. Reply-To — the From address (billjobs@microapple.com) and the Reply-To address (negeja3921@pashter.com) belonged to two completely different domains. This mismatch is a classic Business Email Compromise / phishing pattern: the attacker spoofs a trusted-looking sender but routes any reply to an address they actually control.
Content-Type — multipart/mixed, which tells the receiving mail client that the message contains more than one content block (e.g. a text body plus one or more attachments). The boundary value defines the unique delimiter string the mail server uses to know where each block starts and ends.
Message-ID — a unique identifier generated for tracking/referencing the specific email.
Date — the timestamp the recipient's client displays. Important caveat: this field can be spoofed and should never be trusted in isolation.
Header summary:
Date:          Tue, 26 Jan 2012 01:41:18 (EST)
Return-Path:   billjobs@microapple.com
Received:      emkei.cz (localhost) (93.99.104.210)
Reply-To:      negeja3921@pashter.com
Received-SPF:  fail

<img width="1579" height="858" alt="email 1" src="https://github.com/user-attachments/assets/37308ab6-a435-4333-bfea-4caec4024019" />

OSINT on the Sending Infrastructure:
The originating server, emkei.cz, is a well-known fake mailer / anonymous email service. It's a free web-based tool that lets anyone spoof a "From" address and craft a custom email using an HTML editor, encryption options, and other advanced sending settings. Its appearance in the Received chain — combined with the SPF fail and the microapple.com spoof — confirmed the message was not sent by Apple/Microsoft at all, but forged.

2. Email Body Analysis:
The body itself was a ransom-style threat directed at the Major, demanding $1 billion USD and referencing the missing CoCanDians, with an attachment presented as proof/instructions.
The second part of the message was text/plain, Base64-encoded. Rather than just decoding it to text, I converted the decoded output to hex to inspect the raw byte structure  because a file's name and its actual file type are not the same thing, and attackers routinely disguise one as the other.

<img width="800" height="449" alt="Base64 cyberchef" src="https://github.com/user-attachments/assets/32a2f50b-85fd-43e6-83a3-e542422a63d7" />

3. Attachment Analysis — "PuzzleToCoCanDa.pdf" Is Not a PDF
The very first bytes of the attachment, once viewed in hex, were:
50 4B 03 04
<img width="519" height="243" alt="pdf" src="https://github.com/user-attachments/assets/95b331c1-4754-4e29-b1df-d420c1405225" />

Cross-referencing this against the Gary Kessler File Signature Table shows 50 4B 03 04 (PK\x03\x04) is the magic number for a ZIP archive 
 not a PDF. A genuine PDF should begin with:
25 50 44 46   →   %PDF (HxD app)

<img width="722" height="425" alt="Hiddenfiles" src="https://github.com/user-attachments/assets/8a8a867f-f66d-4417-8be2-1f24a828308c" />

This confirmed the attacker had renamed a ZIP archive with a .pdf extension to make it look harmless and get past casual inspection.
Unpacking the Archive
I saved the payload as attachment.zip and extracted it. Inside was a hidden file, along with what looked like a mix of file types — but extensions alone couldn't be trusted at this point, so every file needed its signature manually verified in HxD.
File 1 — no extension / "looked like Excel": Opened the raw bytes in HxD and found:
FF D8 FF E0

<img width="704" height="420" alt="bound 2 indicate picture jpg" src="https://github.com/user-attachments/assets/20cba2f6-e895-4742-bf55-dc0b66c4e2ef" />

Per Gary Kessler's table, this is the signature for a JPEG image. I renamed the file with a .jpg extension and confirmed it opened correctly as a picture.
File 2: Bytes in HxD again showed the ZIP signature (50 4B 03 04) rather than a standalone document header. Since this didn't behave as a normal single-format file, I used the online viewer at sqrx.com, which supports opening a wide range of file/container formats directly in the browser, to safely inspect its actual contents without relying on a local, potentially vulnerable application.
File 3 — the "PDF": Verified separately to confirm it matched a genuine PDF structure and was the intended lure document.
Note on Office file signatures: it's worth calling out for anyone following this write-up — modern Office formats (.xlsx, .docx, .pptx) are themselves ZIP containers, so seeing PK bytes at the start of a file claiming to be Excel isn't automatically malicious on its own. The suspicious part in this case was the outer file (PuzzleToCoCanDa.pdf) being a ZIP wrapper for a completely different, undisclosed set of documents — that inconsistency between the claimed and actual structure is what mattered.
<img width="752" height="380" alt="gary kessel" src="https://github.com/user-attachments/assets/10fc3900-22d2-4912-a858-3c488875b687" />

 Key Indicators of Compromise (IOCs)
Indicator
Value
Spoofed sending domain
microapple.com
Sending IP
93.99.104.210
Fake mailer service
emkei.cz
Return-Path
billjobs@microapple.com
Reply-To (attacker-controlled)
negeja3921@pashter.com
Malicious attachment (disguised extension)
PuzzleToCoCanDa.pdf (actual type: ZIP)
SPF result
Fail

 Conclusion:
This investigation confirmed the email was a spoofed phishing t, not a legitimate communication. Three  lines of evidence supported this:
The Received-SPF result failed, showing the sending IP was never authorized by the claimed domain.
The From and Reply-To addresses diverged, a hallmark of spoofing meant to redirect victim responses to an attacker-owned inbox.
The sending infrastructure traced back to emkei.cz, a publicly known fake-mailer service used to forge sender identities.
The attachment itself was a layered deception: a file claiming to be a PDF was actually a ZIP archive, which in turn concealed a JPEG, a genuine PDF, and a third document — none of which matched their apparent extensions at a glance. Verifying true file type through magic byte / file signature analysis rather than trusting filenames or extensions was the critical step that unraveled the payload.

 Skills Demonstrated
Manual email header parsing and authentication (SPF/DKIM/DMARC) analysis
Identifying sender spoofing via From/Reply-To/Return-Path inconsistencies
OSINT on suspicious sending infrastructure
Base64 decoding and hex-level file inspection
File signature ("magic number") analysis to detect disguised/renamed malicious files
Safe, methodical handling of a multi-layered phishing attachment

 Hands-on SOC analyst training (MyDFIR-style lab, sourced from BlueTeamLabs Online).
