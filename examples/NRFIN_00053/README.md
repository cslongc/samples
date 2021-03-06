# One_Vote

## Author Information

"Nick Davis" <info@narfindustries.com>

### DARPA performer group

Narf Industries (NRFIN)

## Description:

Institutionally-sponsored popularity contests are common events in the lives of American adolescents. In response to the roar of demanding parents, we've developed One Vote to optimally support the needs of these myriad institutions. 

One Vote will guarantee that: only the most popular adolescent wins the contest, an individual's vote can never be seen by other voters, an individual's vote can never be associated with their name, and each potential voter can only vote one time (although they can change their vote as many times as they want while voting is open).

## Feature List:

- Admin
-- Create Election: create election manager's credentials, define election name, opt to (dis/en)able write-in candidates, define number of winning candidate(s), and max number of total candidates.
-- Open Voting: election manager can start voting period.
-- Close Voting: election manager can end voting period.

- Voting
-- Login: voters or election manager can login to access functions requiring authorization.
-- Register Voter: enter first and last name and get voter ID number.
-- Vote: enter voter ID and last name and make voting choice. If already voted, can change choice.
-- Add Candidate: add a new candidate to the election.

- Results
-- Election status: list number of voters, number of votes, number of candidates
-- Voting Results: list winning candidate(s) along with vote count in order from highest to lowest vote count.
-- Voter Turnout: list number of votes, number of voters, names of all voters
-- Candidate Summary: list number of candidates, names of all candidates with vote counts.

## Vulnerability 1

When a voter submits a vote, they get a voting receipt that contains a confirmation number. In the unpatched version, that confirmation number is the voter's ID xor'd with the first 4 bytes of the flag page. The function get\_next\_vote\_id() contains this vulnerability.

### Generic Class of Vulnerability

Information Leakage

### CWE Classification

CWE-201: Information Exposure Through Sent Data

### Challenges

* Reaching this vulnerability will require the CRS to generate input that will successfully progress through several pre-requisite steps. They have to create the election, add a candidate, add a voter, open the election while logged in as the election manager, and finally vote while logged in as a voter.
* The login process requires the CRS to both use previously entered information generated by the CRS along with information sent by the CB in response to an earlier command.
* There are only a few commands that are valid and those commands are only accepted depending on the current state and in several cases, if the login process has been completed. This means the CRS will need to keep track of state.

### Difficulty

* Discovering is easy
* Proving is medium
* Fixing is easy

## Vulnerability 2

When a voter submits a vote, a vote record is created that contains the voters information as well as the candidates information. The vote record is stored in a hash table where the key is the concatenation of the first and last name of the voter. The key is created using snprintf and a format string that will combine the first and last name, separated by a space, into a single string. In the unpatched version, the format string was accidentally omitted from the call to snprintf, so the voter's first name is used as the format string. This is in election.c:create\_and\_insert\_vote().

### Generic Class of Vulnerability

Buffer Overflow
Uncontrolled Format String

### CWE Classification

CWE-120 Buffer Copy without Checking Size of Input
CWE-121 Stack-based Buffer Overflow
CWE-134 Uncontrolled Format String
CWE-787 Out-of-bounds Write
CWE-788 Access of Memory Location After End of Buffer
CWE-806 Buffer Access Using Size of Source Buffer

### Challenges

* Reaching this vulnerability will require the CRS to generate input that will successfully progress through several pre-requisite steps. They have to create the election, add a candidate, add a voter, open the election while logged in as the election manager, and finally vote while logged in as a voter.
* The login process requires the CRS to both use previously entered information generated by the CRS along with information sent by the CB in response to an earlier command.
* There are only a few commands that are valid and those commands are only accepted depending on the current state and in several cases, if the login process has been completed. This means the CRS will need to keep track of state.
* The data that triggers the vulnerability is entered in the function to register a voter. This is usually in a different state and not closely related to the point at which that voter places a vote. The CRS will have to track the relationship between the new voter's first and last name and this vulnerability. The CRS will have to learn: that the first name needs to be a format string, which collection of format chars to include in the format string, and how many chars to include in the last name. There are a large number of combinations of format chars and string lengths that will cause the CB to crash, but only a small number of combinations that will trigger a POV.

### Difficulty

* Discovering is easy
* Proving is medium
* Fixing is easy
