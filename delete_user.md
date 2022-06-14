delete_user



| import getopt |                                          |
| ------------- | ---------------------------------------- |
|               |                                          |
|               | import boto3,sys                         |
|               | import configparser                      |
|               |                                          |
|               | cf = configparser.ConfigParser()         |
|               | cf.read("../config/test.conf")           |
|               | # if "vdsrcndev" in cf.keys():           |
|               | #     print("success!")                  |
|               | profiles = cf.sections()                 |
|               | # print(profiles)                        |
|               | print("account num:{}".format(len(profiles))) |
|               |                                          |
|               | class deleteUser(object):                |
|               | '''init func'''                          |
|               |                                          |
|               | def __init__(self):                      |
|               |                                          |
|               | self.lists = []                          |
|               | self.param = {}                          |
|               |                                          |
|               | '''                                      |
|               | 描述：调度的主函数                                |
|               | resP：检测入参结果，如果类型不是 bool 说明有报错            |
|               | '''                                      |
|               | def main(self, argv):                    |
|               | # print(argv)                            |
|               | if len(argv) < 1:                        |
|               | sys.exit("\nusage: " + sys.argv[0] + " -h ") |
|               | try:                                     |
|               | opts, args = getopt.getopt(sys.argv[1:], '-a:-u:h', ['account','username','help']) |
|               | # print("print opts={}, args={}".format(opts,args)) |
|               | except Exception as e:                   |
|               | sys.exit("\nusage: " + sys.argv[0] + " -h ") |
|               | print("print opts={}, args={}".format(opts, args)) |
|               |                                          |
|               |                                          |
|               | flag = True                              |
|               | users=[]                                 |
|               | accounts=[]                              |
|               | # print("resp={}".format(resP))          |
|               | for opt_name, opt_value in opts:         |
|               | if opt_name in ('-h','--help'):          |
|               | self.helps()                             |
|               | sys.exit()                               |
|               | if opt_name in ('-u','--username'):      |
|               | self.param['-u'] = opt_value             |
|               | users = self.param['-u'].split(',')      |
|               | if opt_name in ('-a', '--account'):      |
|               | if opt_value == 'all':                   |
|               | accounts = profiles                      |
|               | flag = False                             |
|               | else:                                    |
|               | self.param['-a'] = opt_value             |
|               | accounts = self.param['-a'].split(',')   |
|               | # print("flag={}".format(flag))          |
|               | print("opt_name={} ,opt_value={}".format(opt_name, opt_value)) |
|               |                                          |
|               | print("account={}".format(accounts))     |
|               |                                          |
|               | if flag:                                 |
|               | resP = self.doCheck(self.param)          |
|               | if not isinstance(resP, bool): sys.exit(resP) |
|               | for account in accounts:                 |
|               | print(account)                           |
|               | for username in users:                   |
|               | print(username)                          |
|               | self.deleteLoginProfile(account, username) |
|               | self.deleteAccessKeys(account, username) |
|               | self.deleteSigningCertificates(account, username) |
|               | self.deleteSSHPublicKeys(account, username) |
|               | self.deleteGitCredentials(account, username) |
|               | self.deleteInlineUserPolicies(account, username) |
|               | self.deleteManagedUserPolicies(account, username) |
|               | self.deleteMFADevices(account, username) |
|               | self.removeUserFromGroups(account, username) |
|               | self.deleteUserFun(account, username)    |
|               |                                          |
|               | def doCheck(self, param):                |
|               | try:                                     |
|               | print(type(param))                       |
|               | print(param.keys())                      |
|               | for key1 in ('-u','-a'):                 |
|               | if not key1 in param.keys():             |
|               | return "[Error]: {0} Must be by parameter".format(key1) |
|               | except KeyError as e:                    |
|               | return "[Error]: Parameter {0} error".format(str(e)) |
|               | return True                              |
|               |                                          |
|               | def helps(self):                         |
|               | print("\nscript options explain: \       |
|               | \n\t -u <username>                                   要删除的IAM用户id；\ |
|               | \n\t -a <account\|all>                                aws账号本地profile名称,例如vdsrcndev,多个账号用','分开,all表示删除../config/test.conf中所有账号下用户") |
|               |                                          |
|               |                                          |
|               | def deleteLoginProfile(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | try:                                     |
|               | iam.delete_login_profile(                |
|               | UserName=username                        |
|               | )                                        |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteAccessKeys(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | accesskeys = []                          |
|               | try:                                     |
|               | response = iam.list_access_keys(         |
|               | UserName=username                        |
|               | )                                        |
|               | if response:                             |
|               | for i in response['AccessKeyMetadata']:  |
|               | accesskeys.append(i['AccessKeyId'])      |
|               | # print(accesskeys)                      |
|               | for accesskey in accesskeys:             |
|               | iam.delete_access_key(                   |
|               | UserName=username,                       |
|               | AccessKeyId=accesskey                    |
|               | )                                        |
|               | print("delete accesskey : {} successfully!".format(accesskey)) |
|               |                                          |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteSigningCertificates(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | certificates = []                        |
|               | try:                                     |
|               | response = iam.list_signing_certificates( |
|               | UserName=username                        |
|               | )                                        |
|               | if response:                             |
|               | for i in response['Certificates']:       |
|               | certificates.append(i['CertificateId'])  |
|               | for certificate in certificates:         |
|               | iam.delete_signing_certificate(          |
|               | UserName=username,                       |
|               | CertificateId=certificate                |
|               | )                                        |
|               | print("delete certificateid :  {} successfully!".format(certificate)) |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteSSHPublicKeys(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | sshPublicKeys = []                       |
|               | try:                                     |
|               | response = iam.list_ssh_public_keys(     |
|               | UserName=username                        |
|               | )                                        |
|               | # print(response.keys())                 |
|               | if 'SSHPublicKeys' in response.keys():   |
|               | # print("deleteSSHPublicKeys true")      |
|               | for i in response['SSHPublicKeys']:      |
|               | sshPublicKeys.append(i['SSHPublicKeyId']) |
|               | for sshPublicKey in sshPublicKeys:       |
|               | iam.delete_ssh_public_key(               |
|               | UserName=username,                       |
|               | SSHPublicKeyId=sshPublicKey              |
|               | )                                        |
|               | print("delete sshpublickey : {} successfully!".format(sshPublicKey)) |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteGitCredentials(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | ServiceSpecificCredentials = []          |
|               | try:                                     |
|               | response = iam.list_service_specific_credentials( |
|               | UserName=username                        |
|               | )                                        |
|               | if 'ServiceSpecificCredentials' in response.keys(): |
|               | for i in response['ServiceSpecificCredentials']: |
|               | ServiceSpecificCredentials.append(i['ServiceSpecificCredentialId']) |
|               | for ServiceSpecificCredential in ServiceSpecificCredentials: |
|               | iam.delete_service_specific_credential(  |
|               | UserName=username,                       |
|               | ServiceSpecificCredentialId=ServiceSpecificCredential |
|               | )                                        |
|               | print("delete ServiceSpecificCredentialId : {} successfully!".format(ServiceSpecificCredential)) |
|               |                                          |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteMFADevices(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | devices = []                             |
|               | try:                                     |
|               | response = iam.list_mfa_devices(         |
|               | UserName=username                        |
|               | )                                        |
|               | if 'MFADevices' in response.keys():      |
|               | for i in response['MFADevices']:         |
|               | devices.append(i['SerialNumber'])        |
|               | for device in devices:                   |
|               | iam.deactivate_mfa_device(               |
|               | UserName=username,                       |
|               | SerialNumber=device                      |
|               | )                                        |
|               | print("delete device :  {} successfully!".format(device)) |
|               |                                          |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteInlineUserPolicies(self, account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | policynames = []                         |
|               | try:                                     |
|               | response = iam.list_user_policies(       |
|               | UserName=username                        |
|               | )                                        |
|               | if 'PolicyNames' in response.keys():     |
|               | for i in response:                       |
|               | policynames.append(i['PolicyNames'])     |
|               | for policyname in policynames:           |
|               | iam.delete_user_policy(                  |
|               | UserName=username,                       |
|               | PolicyName=policyname                    |
|               | )                                        |
|               | print("delete userpolicy : {} successfully!".format(policyname)) |
|               |                                          |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteManagedUserPolicies(self,account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | policyarns = []                          |
|               | try:                                     |
|               | response = iam.list_attached_user_policies( |
|               | UserName=username                        |
|               | )                                        |
|               | if 'AttachedPolicies' in response.keys(): |
|               | for i in response['AttachedPolicies']:   |
|               | policyarns.append(i['PolicyArn'])        |
|               | for policyarn in policyarns:             |
|               | iam.detach_user_policy(                  |
|               | UserName=username,                       |
|               | PolicyArn=policyarn                      |
|               | )                                        |
|               | print("detach policyarn: {} from user: {] successfully!".format(policyarn,username)) |
|               |                                          |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def removeUserFromGroups(self,account,username): |
|               | # print("account={},username={}".format(account.username)) |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | groups = []                              |
|               | try:                                     |
|               | response = iam.list_groups_for_user(     |
|               | UserName=username                        |
|               | )                                        |
|               | if 'Groups' in response.keys():          |
|               | for i in response['Groups']:             |
|               | groups.append(i['GroupName'])            |
|               | for group in groups:                     |
|               | iam.remove_user_from_group(              |
|               | GroupName=group,                         |
|               | UserName=username                        |
|               | )                                        |
|               | print("remove user: {} from group: {} successfully!".format(username,group)) |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | def deleteUserFun(self,account,username): |
|               | b3 = boto3.session.Session(              |
|               | profile_name=account                     |
|               | )                                        |
|               | iam = b3.client('iam')                   |
|               | try:                                     |
|               | response = iam.delete_user(              |
|               | UserName=username                        |
|               | )                                        |
|               | # print(response)                        |
|               | if 'ResponseMatadata' in response.keys(): |
|               | print("delete user: {} from account: {} successfully!".format(username, account)) |
|               | except Exception as ex:                  |
|               | return "Error occurs as following : {}".format(ex) |
|               | return True                              |
|               |                                          |
|               | # TODO 入口                                |
|               |                                          |
|               | if __name__ == '__main__':               |
|               | fun = deleteUser()                       |
|               | fun.main(sys.argv[1:])                   |