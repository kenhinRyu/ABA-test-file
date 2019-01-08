0                 01CBA       Homepal Technology Pty Ltd      301500withdraw    080119                                    
1013-160193784111 500000000001Smith Joan Emma                 Homepal userID    063-010013626730Homepal         00000000
7999-999            000000000100000000010000000000                        000001                                        

------------------------------------------------------------------------------------------------------------------------
123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890
0        1         2         3         4         5         6         7         8         9        10        11        12
------------------------------------------------------------------------------------------------------------------------
第1-3行为ABA文件的有效内容，第5-8行是“格尺”。以下为解释说明，实际ABA文件中不需要第4行（包括第四行）以后的内容。

第一行：descriptive record
位置      长度      内容                              解释
1         1        ‘0’                  记录类型，此处永远是‘0’，代表此行是descriptive record
2-18      17        空                             未被使用
19-20     2       ‘01’                             永远是‘1’
21-23     3       ‘CBA’                         银行名称，此处永远是‘CBA’
24-40     7         空                              未被使用
31-56     26   左对齐，填充空格         用户名称，此处永远是：‘Homepal Technology Pty Ltd’ 
57-62     6     右对齐，填充0                  用户码，此处永远是：‘301500’
63-74     12   左对齐，填充空格                  提现说明：此处暂定‘withdraw’
75-80     6    DDMMYY, 的日期格式，0填充       后台用户点击生成ABA文件的日期，例：’080119‘，表示2019年1月8号。
81-120    40        空                              未被使用

第二行：detail record format （此处可以是多行（500行以内），每行代表一笔提现交易记录）
位置      长度      内容                              解释
1         1        ‘1’                  记录类型，此处永远是‘1’，代表此行是detail record format
2-8       7      XXX-XXX                          收款用户的BSB号码
9-17      9       右对齐                收款账户的账号，（如果长度大于9位，删去连字符hypens）
18        1        留空                            此处留空
19-20     2        ‘50’                             交易代码
21-30     10   右对齐，填充‘0’。交易金额        金额显示到分不需要小数点：例：一百澳元，显示为0000010000
31-62     32    左对齐，填充空格                  收款账户名 （用户在Homepal系统中填写的姓名：‘姓. 名’）
63-80     18   左对齐，填充空格                         Homepal系统内的用户ID
81-87      7    XXX-XXX                              支付失败时的退款BSB：永远是：‘063-010’
88-96      9       右对齐，填充空格                           支付失败时的退款账号：永远是：‘013626730’
97-112    16       左对齐，填充空格                   汇款人姓名：‘Homepal’
113-120   8       右对齐，填充‘0’                   此处永远填充‘00000000’

第三行：File total record
位置      长度      内容                              解释
1         1        ‘7’                  记录类型，此处永远是‘7’，代表此行是file total record
2-8       7      '999-999'                       BSB number: ''
9-20      12       留空                             此处留空
21-30     10     右对齐，补‘0’                 net total amount 净额（此处等于总提现额，因为此处现在没有收款功能）
31-40     10     右对齐，补‘0’                   credit total amount 总提现额（detail record format 里的金额总和）
41-50     10     右对齐，补‘0’                 net debit total amount 总收款额 （此处永远是‘0000000000’）
51-74     24       留空                              未被使用
75-80     6        右对齐，补‘0’                  detail record 的数量
81-120    40        空                              未被使用


EOF should be after 120th chracter of line 3. Everything below that is comments. Lines 6-7 are a ruler of sorts, showing
the location of characters 1 - 120.

File must be CR/LF delimited fixed width (each line must be 120 characters).

Direct Entry User ID codes and APCA Bank Codes (as used in descriptive records) are:

    Code    User ID     Bank name
    ----    -------     ---------
    CBA     301500      Commonwealth ComBiz
    CBA     (ignored)   Commonwealth NetBank
    NAB     (ignored)   nab personal
    ANZ     (ignored)   ANZ personal
    WPC     037819      Westpac personal

Each record type contains a number of fields described below (lines 44-113). Headings for those descriptions are:

    S.P = Start position (1..120)
    E.P = End position (1..120)
    LEN = Length of field (1..120)
    T   = Field Type being one of:
                    A = Alpha                A-Za-z0-9
                    B = BECS character set   A-Za-z0-9^_[]',?;:=#/.*()&%!$ @+-
                    C = Limited characters   A-Za-z0-9 -
                    D = Delimited numeric    0-9-
                    N = Unsigned numeric     0-9
                    F = Fixed Value as described in DESCRIPTION
    A   = Alignment within field size (L = Left, R = Right)
    F   = Fill if length of data less than size of field (Z = zero filled, S = space filled)

------------------------------------------------------------------------------------------------------------------------
Descriptive Record (Record Type = 0)
------------------------------------------------------------------------------------------------------------------------
Must be first record in file. Must be precisely one such record in the file.

S.P  E.P  LEN T A F NAME                DESCRIPTION
  1    1    1 F - - Record Type         Must be 0 for descriptive record.
  2    8    7 D L S Ext:BSB             BSB of funds account (formatted 000-000) OR blank (ignored by WPC; APCA
                                        specification requires blank).
  9   17    9 C R S Ext:Account Number  Account number of fund account (inc leading zeros) OR blank (ignored by WPC;
                                        APCA specification requires blank).
 18   18    1 F - S Reserved            Must be a single blank space.
 19   20    2 N R Z Sequence Number     Generally 01. Sequence number of file in batch (starting from 01). Batches to
                                        be used where number of detail records exceeds maximum per file of 500.
 21   23    3 A - - Bank Name           Three letter APCA abbreviation for bank (CBA, NAB, ANZ, WPC). See APCA
                                        publication entitled BSB Numbers in Australia.
 24   30    7 F - S Reserved            Must be seven blank spaces.
 31   56   26 B L S User Name           The name of the user supplying the file. Some banks must match account holder
                                        or be specified as "SURNAME Firstname". Must not be blank. APCA says it should
                                        be "User preferred name".
 57   62    6 N R Z DE User ID          Direct Entry user ID where allocated. Required for direct debits. For internet
                                        banking use CBA: 301500, WPC: 037819, ignored by NAB and ANZ. APCA requires this
                                        to be BECS User Identification Number.
 63   74   12 B L S File Description    A description of the contents of the file. Ignored by CBA.
 75   80    6 N - Z Processing Date     Date to process transactions as DDMMYY.
 81   84    4 A L S Ext:Processing Time Time to process transactions as 24 hr HHmm or all spaces. APCA specification
                                        requires blank.
 85  120   36 F - S Reserved            Must be thirty six blank spaces.

------------------------------------------------------------------------------------------------------------------------
Detail Record (Record Type = 1)
------------------------------------------------------------------------------------------------------------------------
Must be at least one such record in the file. Must not be first or last record in the file. Must not be more than 500
such records per file (though that limit may be increased by certain financial institutions).

S.P  E.P  LEN T A F NAME                DESCRIPTION
  1    1    1 F - - Record Type         Must be 1 for detail record.
  2    8    7 D L S BSB                 BSB of target account (formatted 000-000).
  9   17    9 C R S Account Number      Account number of target account (inc leading zeros if part of account number).
                                        If account number exceeds nine characters, strip any hyphens.
 18   18    1 A L S Indicator           Must be a blank space or one of:
                                                N - new or varied BSB number or name details (?)
                                                T - drawing under a Transaction Negotiation Authority
                                                W - dividend payment to a resident of a country with a double tax
                                                    agreement
                                                X - dividend payment to a resident of any other country
                                                Y - interest payment to a non-resident of Australia
                                        W, X and Y require that a withholding tax amount be specified.
 19   20    2 N R Z Transaction Code    Must be valid industry standard transaction code (BECS Procedures 4.6).
                                        Available codes are:
                                                13 - externally initiated debit
                                                50 - externally initiated credit - normally what is required
                                                51 - Australian government security interest
                                                52 - Basic family payments/additional family payment 
                                                53 - Payroll payment
                                                54 - Pension payment
                                                55 - Allotment
                                                56 - Dividend
                                                57 - Debenture/note interest
 21   30   10 N R Z Transaction Amount  Total amount of this transaction as zero-padded number of cents (unsigned).
 31   62   32 B L S Account Name        Name target account is held in, normally as "SURNAME. First Second Names". Must
                                        not be all blanks.
 63   80   18 B L S Lodgement Reference Reference (narration) that appears on target's bank statement.
 81   87    7 D L S Trace BSB           BSB of fund (source) account (formatted 000-000).
 88   96    9 C R S Trace Account Num   Account number of fund (source) account (inc leading zeros if part of number).
                                        If account number exceeds nine characters, strip any hyphens.
 97  112   16 B L S Remitter Name       Name of remitter (appears on target's bank statement; must not be blank but
                                        some banks will replace with name fund account is held in).
113  120    8 N R Z Withholding amount  Amount of withholding tax in unsigned cents or all zeros. If not zero then will
                                        cause Indicator field to be ignored and tax to be withheld.

------------------------------------------------------------------------------------------------------------------------
File Total Record (Record Type = 7)
------------------------------------------------------------------------------------------------------------------------
Must be last record in each file of batch. Must be precisely one such record in each file. EOF should occur at the end
of this line (no CRLF).

S.P  E.P  LEN T A F NAME                DESCRIPTION
  1    1    1 F - - Record Type         Must be 7 for batch control record.
  2    8    7 F - - BSB                 Must be "999-999".
  9   20   12 F - S Reserved            Must be twelve blank spaces.
 21   30   10 N R Z Batch Net Total     Total of credits minus total of debits in batch as zero-padded number of cents.
 31   40   10 N R Z Batch Credits Total Total of credits in batch as zero-padded number of cents. Some banks permit
                                        this to be ignored by placing all zeros or all spaces. APCA requires this.
 41   50   10 N R Z Batch Debits Total  Total of debits in batch as zero-padded number of cents. Some banks permit
                                        this to be ignored by placing all zeros or all spaces. APCA requires this.
 51   74   24 F - S Reserved            Must be twenty four blank spaces.
 75   80    6 N R Z Number of records   Must be the total number of detail records in the batch, zero-padded.
 81  120   40 F - S Reserved            Must be forty blank spaces.

------------------------------------------------------------------------------------------------------------------------
FORMAL SPECIFICATION
------------------------------------------------------------------------------------------------------------------------
It turns out there is a formal specification for .aba files. The file type is formally known as a BECS DE file (i.e.
Bulk Electronic Clearing System Direct Entry file). The file format specification is published by the Australian
Payments Clearing Association in Appendix C2 (pages 78 - 85) and the character set in Appendix C7 (page 86) of:
http://www.apca.com.au/docs/payment-systems/becs_procedures.pdf
 
------------------------------------------------------------------------------------------------------------------------
REFERENCES
------------------------------------------------------------------------------------------------------------------------
http://www.cemtexaba.com/aba-format/cemtex-aba-file-format-details.html
http://ddkonline.blogspot.com.au/2009/01/aba-bank-payment-file-format-australian.html
http://www.brad-smith.info/blog/archives/405
http://www.anz.com/Documents/AU/corporate/clientfileformats.pdf

------------------------------------------------------------------------------------------------------------------------
ABOUT THIS FILE
------------------------------------------------------------------------------------------------------------------------
Author:  Michael Cordover
Email:   aba@mjec.net
Created: 2013-04-07
Updated: 2013-04-07
Version: 1.1
Licence: CC-BY 3.0 AU <http://creativecommons.org/licenses/by/3.0/au/>
