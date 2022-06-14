add_user



|               |                                          |
| ------------- | ---------------------------------------- |
| import string |                                          |
|               | import random                            |
|               | import urllib                            |
|               | from email.mime.application import MIMEApplication |
|               | from email.mime.multipart import MIMEMultipart |
|               | from email.mime.text import MIMEText     |
|               |                                          |
|               | import botostubs                         |
|               | import json                              |
|               |                                          |
|               | import boto3                             |
|               | import boto3.s3                          |
|               | import xlrd                              |
|               | import xlwt                              |
|               | import time                              |
|               | import datetime                          |
|               |                                          |
|               |                                          |
|               | def default(o):                          |
|               | if type(o) is datetime.date or type(o) is datetime.datetime: |
|               | return o.isoformat()                     |
|               |                                          |
|               | def set_style(name, height, bold=False): |
|               | style = xlwt.XFStyle()                   |
|               | font = xlwt.Font()                       |
|               | font.name = name                         |
|               | font.bold = bold                         |
|               | font.color_index = 4                     |
|               | font.height = height                     |
|               | style.font = font                        |
|               | return style                             |
|               |                                          |
|               | class ExcelHandler(object):              |
|               | def __init__(self, fileName):            |
|               | self.fileName = fileName                 |
|               | self.f = xlwt.Workbook(encoding="utf-8") |
|               |                                          |
|               | def addSheet(self, sheetName, row0):     |
|               | self.sheetName = sheetName               |
|               | self.sheet1 = self.f.add_sheet(sheetName, cell_overwrite_ok=True) |
|               | self.row0 = row0                         |
|               | for i in range(0, len(self.row0)):       |
|               | self.sheet1.write(0, i, row0[i] ,set_style('Regular', 220, True)) |
|               | self.row = 1                             |
|               |                                          |
|               | def writeToExcel(self, data):            |
|               | self.data = data                         |
|               | if len(data) > 1:                        |
|               | for i in range(len(data)):               |
|               | value = data[i]                          |
|               | self.sheet1.write(self.row, i , value)   |
|               | self.f.save(self.fileName)               |
|               | self.row = self.row + 1                  |
|               |                                          |
|               | def list_groups(project):                |
|               | session = boto3.Session(profile_name=project) |
|               | iam = session.client('iam')              |
|               | group_list = [groups['GroupName'] for groups in iam.list_groups()['Groups']] |
|               | return group_list                        |
|               |                                          |
|               | def add_users_to_group(project, username, groupname): |
|               | session = boto3.Session(profile_name=project) |
|               | iam = session.client('iam')              |
|               | iam.add_user_to_group(                   |
|               | GroupName=groupname,                     |
|               | UserName=username                        |
|               | )                                        |
|               |                                          |
|               | src_digits = string.digits              #string_数字 |
|               | src_uppercase = string.ascii_uppercase  #string_大写字母 |
|               | src_lowercase = string.ascii_lowercase  #string_小写字母 |
|               | symbols = '!@#$%&*+='                    |
|               | def getpwd():                            |
|               | for i in range(8):                       |
|               | # 随机生成数字、大写字母、小写字母的组成个数（可根据实际需要进行更改）     |
|               | digits_num = random.randint(1, 6)        |
|               | uppercase_num = random.randint(1, 8 - digits_num - 1) |
|               | lowercase_num = 8 - (digits_num + uppercase_num) |
|               |                                          |
|               |                                          |
|               | # 生成字符串                                  |
|               | password = random.sample(src_digits, digits_num) + random.sample(src_uppercase, uppercase_num) + random.sample( |
|               | src_lowercase, lowercase_num) + random.sample(symbols, 1) |
|               |                                          |
|               | # 打乱字符串                                  |
|               | random.shuffle(password)                 |
|               |                                          |
|               | # 列表转字符串                                 |
|               | new_password = ''.join(password)         |
|               | return new_password                      |
|               |                                          |
|               | def create_user(project, username):      |
|               | session = boto3.Session(profile_name=project) |
|               | iam = session.client('iam')              |
|               |                                          |
|               | user_data = []                           |
|               | console_pwd = getpwd()                   |
|               |                                          |
|               | iam.create_user(                         |
|               | UserName=username                        |
|               | )                                        |
|               | iam.create_login_profile(                |
|               | UserName=username,                       |
|               | Password=console_pwd,                    |
|               | PasswordResetRequired=True               |
|               | )                                        |
|               | response = iam.create_access_key(        |
|               | UserName=username                        |
|               | )                                        |
|               |                                          |
|               | # user_data.append([project, username,  response['AccessKey']['AccessKeyId'], |
|               | #                   response['AccessKey']['SecretAccessKey']]) |
|               | user_data.append([project, username, console_pwd, response['AccessKey']['AccessKeyId'], |
|               | response['AccessKey']['SecretAccessKey']]) |
|               | return user_data                         |
|               |                                          |
|               | def remove_user_from_group(project,username,groupname): |
|               | session = boto3.Session(profile_name=project) |
|               | iam = session.client('iam')              |
|               | iam.remove_user_from_group(              |
|               | GroupName=groupname,                     |
|               | UserName=username                        |
|               | )                                        |
|               |                                          |
|               | def delete_user(project,username):       |
|               | session = boto3.Session(profile_name=project) |
|               | iam = session.client('iam')              |
|               |                                          |
|               | iam.delete_login_profile(                |
|               | UserName=username                        |
|               | )                                        |
|               |                                          |
|               | iam.delete_user(                         |
|               | UserName=username                        |
|               | )                                        |
|               |                                          |
|               | def SendEmail(username, account, filename): |
|               | source = 'noreply.srcn@samsungcloud.tv'  |
|               | recipient = username + '@samsung.com'    |
|               | subject = "IAM User Info for AWS Account" |
|               | consoleurl = "https://signin.amazonaws.cn" |
|               | content = 'Hello,\n\n The attachment is your IAM User Info for the following accounts:\n {}\n Please login from url:{}'.format(account, consoleurl) |
|               |                                          |
|               | account_ses = boto3.client('ses',        |
|               | aws_access_key_id='***',                 |
|               | aws_secret_access_key='***',             |
|               | region_name='us-east-1'                  |
|               | )                                        |
|               | msg = MIMEMultipart('mixed')             |
|               | msg['Subject'] = subject                 |
|               | msg['From'] = source                     |
|               | msg['To'] = recipient                    |
|               |                                          |
|               | # mail body                              |
|               | msg.attach(MIMEText(content, 'html', 'utf-8')) |
|               |                                          |
|               | # add attchment                          |
|               | attachment_filename = 'userinfo_{}.xls'.format(username) |
|               | att = MIMEApplication(open('./{}'.format(filename), 'rb').read(), 'utf-8') |
|               | att["Content-Type"] = 'application/actet-stream' |
|               | att.add_header('Content-Disposition', 'attachment', filename=attachment_filename) |
|               | # att['Content-Disposition'] = 'attachment; filename = attachment_filename' |
|               |                                          |
|               | msg.attach(att)                          |
|               |                                          |
|               | # Create de SMTP Object to Send          |
|               | account_ses.send_raw_email(Source=source, |
|               | RawMessage={                             |
|               | 'Data': msg.as_string()                  |
|               | # this generates all the headers and stuff for a raw mail message |
|               | })                                       |
|               | print(content)                           |
|               |                                          |
|               | if __name__ == '__main__':               |
|               |                                          |
|               | Now = time.strftime('%y%m%d-%H%M', time.localtime(time.time())) |
|               | FileName = 'user_data_{}.xls'.format(Now) |
|               |                                          |
|               | excelHanlder = ExcelHandler(FileName)    |
|               | project_list = ['vdsrcntnc']             |
|               | # project_list_prd = ['vdsrcntncdev', 'vdsrcnbrowserdev','vdsrcntncstg', 'vdsrcnbrowserstg','vdsrcntnc', 'vdsrcnbrowser'] |
|               | users = [                                |
|               | 'ws.kim'                                 |
|               | ]                                        |
|               | user_info = ['Account', 'UserName', 'ConsolePwd', 'AccesskeyId', 'SecretAccesskey'] |
|               | # user_info = ['Account', 'UserName', 'AccesskeyId', 'SecretAccesskey'] |
|               | excelHanlder.addSheet('user', user_info) |
|               |                                          |
|               | # for project in project_list:           |
|               | #     for user in users:                 |
|               | #         user_data = create_user(project, user) |
|               | #         add_users_to_group(project, user, "SRCN_Infra_Admin") |
|               | #         add_users_to_group(project, user, "ConsoleUsers") |
|               | #         for data in user_data:         |
|               | #             excelHanlder.writeToExcel(data) |
|               | #             print("successfully!")     |
|               | #         SendEmail(project, user, user_data, FileName) |
|               |                                          |
|               |                                          |
|               | for user in users:                       |
|               | accounts = []                            |
|               | for project in project_list:             |
|               | accounts.append(project)                 |
|               | user_data = create_user(project, user)   |
|               | # add_users_to_group(project, user, "SRCN_Infra_Admin") |
|               |                                          |
|               | # add_users_to_group(project, user, "SRCN_Service_Develop") |
|               | add_users_to_group(project, user, "ConsoleUsers") |
|               | add_users_to_group(project, user, "Service_Readonly") |
|               | # add_users_to_group(project, user, "HQ_Infra_Readonly") |
|               | # add_users_to_group(project, user, "Bespin_OpsNow") |
|               | # add_users_to_group(project, user, "ProgramUsers") |
|               | for data in user_data:                   |
|               | excelHanlder.writeToExcel(data)          |
|               | print("successfully!")                   |
|               | print(accounts)                          |
|               | SendEmail(user, accounts, FileName)      |
|               |                                          |