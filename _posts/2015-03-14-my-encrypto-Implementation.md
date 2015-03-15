---
layout: post
title: 我所使用的加解密实现 
---

在过往的开发过程中,肯定是离不了加密解密这一系列的操作的,
我所用到的多是基于OpenSSL库实现的RSA的非对称加密和3dES的对称算法,
但从最近使用过的一开发使用中发现在64位设备上会有空指针出现的问题而造成的崩溃,
已经严重影响到APP得正常运行,所以非常有必要整理一份新的加解密实现.......
幸运的是终于在GitHub上找到一套靠谱的算法实现---

###先说说公私钥的生成:

通常我们在CS模式中用到的RSA对称加密,都需要在两端各保留一套公私钥:
我们可以通过keytool生成的一对秘钥,但是繁琐的命令不便于记忆实在麻烦,我这里有一个用java实现的密钥对方法,可以很快捷的生成.
代码如下:


	import java.io.BufferedReader;
	import java.io.File;
	import java.io.FileOutputStream;
	import java.io.InputStreamReader;
	import java.security.KeyPair;
	import java.security.KeyPairGenerator;
	import java.security.NoSuchAlgorithmException;
	import java.security.SecureRandom;
	import java.security.Security;
	import java.security.interfaces.RSAPrivateKey;
	import java.security.interfaces.RSAPublicKey;
	import java.security.spec.PKCS8EncodedKeySpec;
	import java.security.spec.X509EncodedKeySpec;
	import java.text.DecimalFormat;

	public class GeneralKeyPairs {

		public static String pubPath = "keysUnreg/pub/";
		public static String priPath = "keysUnreg/pri/";

		public static int count = 0;

		private static final DecimalFormat df = new DecimalFormat("0000000000");



		public static KeyPair generateKeyPair() throws Exception {
			KeyPairGenerator keyPairGen= null;    
			try {    
				keyPairGen= KeyPairGenerator.getInstance("RSA");    
			} catch (NoSuchAlgorithmException e) {    
				e.printStackTrace();    
			}    
			keyPairGen.initialize(1024, new SecureRandom());    
			KeyPair keyPair= keyPairGen.generateKeyPair();    
			return keyPair;
		}

		public static void saveX509PublicKey(String path, RSAPublicKey pubKey)
			throws Exception {
				FileOutputStream public_file_out = null;
				byte pub_key_bs[] = null;
				X509EncodedKeySpec spec = null;
				try{
					public_file_out = new FileOutputStream(path);
					pub_key_bs = pubKey.getEncoded();
					spec = new X509EncodedKeySpec(pub_key_bs);
					public_file_out.write(spec.getEncoded());
					public_file_out.close();
					public_file_out = null;
					spec = null;
					pub_key_bs = null;
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
					if (null != public_file_out) {
						public_file_out.close();
						public_file_out = null;
					}
					spec = null;
					pub_key_bs = null;
				}
			}

		public static void savePKCS8PrivateKey(String path, RSAPrivateKey priKey)
			throws Exception {
				FileOutputStream private_file_out = null;
				byte pri_key_bs[] = null;
				PKCS8EncodedKeySpec spec = null;
				try{
					private_file_out = new FileOutputStream(path);
					pri_key_bs = priKey.getEncoded();
					spec = new PKCS8EncodedKeySpec(pri_key_bs);
					private_file_out.write(spec.getEncoded());
					private_file_out.close();
					private_file_out = null;
					spec = null;
					pri_key_bs = null;
				} catch (Exception e) {
					e.printStackTrace();
				} finally {
					if (null != private_file_out) {
						private_file_out.close();
						private_file_out = null;
					}
					spec = null;
					pri_key_bs = null;
				}
			}


		public static void saveKeyPairs_S(String filename) throws Exception{

			String filePub =  pubPath+filename+".pub";
			String filePri =  priPath+filename+".pri";

			File pubDir = new File(pubPath);
			pubDir.mkdirs();

			File priDir = new File(priPath);
			priDir.mkdirs();

			KeyPair keyPair = GeneralKeyPairs.generateKeyPair();
			RSAPublicKey pubKey = (RSAPublicKey) keyPair.getPublic();
			//        System.out.println("保存公钥:"+filePub);
			GeneralKeyPairs.saveX509PublicKey(filePub,pubKey);
			RSAPrivateKey priKey = (RSAPrivateKey) keyPair.getPrivate();
			//        System.out.println("保存私钥:"+filePri);
			GeneralKeyPairs.savePKCS8PrivateKey(filePri, priKey);
			count++;
		}
		public static void saveKeyPairs_P(String start,String end,String append) throws Exception{

			for(int i=Integer.parseInt(start);i<=Integer.parseInt(end);i++){
				saveKeyPairs_S(append+"20140328"+df.format(i));
			}

		}

		/**
		 * @param args
		 * args[0]  
		 *    -s : 生成1对密钥
		 *    -p : 批量生成密钥对
		 * args[1]
		 *    密钥存储文件名（args[0]=-s）
		 *    起始密钥名（args[0]=-p）
		 * args[2]
		 *    终止密钥名（args[0]=-p）
		 * args[3]   
		 *    密钥名头（args[0]=-p）
		 * args[4]
		 *    保存目录
		 * @throws Exception 
		 */

		public static void main(String[] args) throws Exception {
			// TODO Auto-generated method stub

			System.out.println("1：生成1对密钥  2：批量生成密钥:");
			BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

			String type = in.readLine();

			if(type.trim().equals("1")){
				System.out.println("生成1对密钥=>密钥名称:");
				in = new BufferedReader(new InputStreamReader(System.in));

				String filename = in.readLine();
				saveKeyPairs_S(filename);
			}else if(type.trim().equals("2")){
				System.out.println("批量生成密钥=>起始区间(int):");
				in = new BufferedReader(new InputStreamReader(System.in));
				String start = in.readLine();
				System.out.println("批量生成密钥=>终止区间(int):");
				in = new BufferedReader(new InputStreamReader(System.in));
				String end = in.readLine();
				System.out.println("批量生成密钥=>密钥名称头部:");
				in = new BufferedReader(new InputStreamReader(System.in));
				String head = in.readLine();

				saveKeyPairs_P(start,end,head);
			}
			System.out.println("成功生成密钥对儿"+count+"个，详情请见目录："+pubPath);
			//        if(args==null||args.length<2){
			//            System.out.println("输入参数有吴!");
			//        }else 
			//        if(args.length>=2){
			//            if(args.length==5){
			//                path = args[4];
			//            }
			//            if(args[0].equals("-s")){
			//                saveKeyPairs_S(args[1]);
			//            }
			//            if(args[0].equals("-p")){
			//                if(args.length>=4){
			//                    saveKeyPairs_P(args[1],args[2],args[3]);
			//                }else{
			//                    System.out.println("输入参数有吴!");
			//                }
			//            }
			//        }
		}

	}

###再说说如何在代码里使用公私钥:
在上面的代码中,公私钥是以文件的形式存在的(公钥:*.pub|私钥:*.pri),如果为了编码方便或者不想把秘钥文件放到自己的APP中,
方法如下(java实现):

	import java.io.BufferedReader;  
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.IOException;  
	import java.io.InputStream;  
	import java.io.InputStreamReader;  
	import java.math.BigInteger;
	import java.security.InvalidKeyException;  
	import java.security.KeyFactory;  
	import java.security.KeyPair;  
	import java.security.KeyPairGenerator;  
	import java.security.NoSuchAlgorithmException;  
	import java.security.PublicKey;
	import java.security.SecureRandom;  
	import java.security.interfaces.RSAPrivateKey;  
	import java.security.interfaces.RSAPublicKey;  
	import java.security.spec.InvalidKeySpecException;  
	import java.security.spec.PKCS8EncodedKeySpec;  
	import java.security.spec.X509EncodedKeySpec;  
	import javax.crypto.BadPaddingException;  
	import javax.crypto.Cipher;  
	import javax.crypto.IllegalBlockSizeException;  
	import javax.crypto.NoSuchPaddingException;  
	import sun.misc.BASE64Decoder;  
	import sun.misc.BASE64Encoder;  

	public class RSAEncrypt {  

		/*
		 * 从私钥文件中读取私钥信息
		 */
		public void readPKCS8PrivateKey(String path)
			throws Exception {
				FileInputStream private_file_in = null;
				byte pri_key_bs[] = null;
				RSAPrivateKey private_key = null;
				PKCS8EncodedKeySpec spec = null;
				KeyFactory kfac = null;
				try{
					private_file_in = new FileInputStream(path);
					pri_key_bs = new byte[1024];
					private_file_in.read(pri_key_bs);
					spec = new PKCS8EncodedKeySpec(pri_key_bs);
					kfac = KeyFactory.getInstance("RSA");//RSA/ECB/PKCS1Padding
					//KeyFactory kfac = KeyFactory.getInstance("RSA/ECB/PKCS1Padding");
					private_key = (RSAPrivateKey) kfac.generatePrivate(spec);
					private_file_in.close();
					private_file_in = null;
					kfac = null;
					spec = null;
					pri_key_bs = null;
				} catch (Exception e) {

				} finally {
					if (null != private_file_in) {
						private_file_in.close();
						private_file_in = null;
					}
					kfac = null;
					spec = null;
					pri_key_bs = null;
				}
				this.privateKey = private_key;
			}
	
	
		/*
		 * 从公钥文件中读取公钥信息
		 */
		public RSAPublicKey readX509PublicKey(String path) throws Exception {
			FileInputStream public_file_in = null;
			byte pub_key_bs[] = null;
			X509EncodedKeySpec spec = null;
			KeyFactory kfac = null;
			RSAPublicKey public_key = null;
			try{
				public_file_in = new FileInputStream(path);
				pub_key_bs = new byte[1024];
				public_file_in.read(pub_key_bs);
				spec = new X509EncodedKeySpec(pub_key_bs);
				kfac = KeyFactory.getInstance("RSA");//RSA/ECB/PKCS1Padding
				//KeyFactory kfac = KeyFactory.getInstance("RSA/ECB/PKCS1Padding");
				public_key = (RSAPublicKey) kfac.generatePublic(spec);
				public_file_in.close();
				public_file_in = null;
				kfac = null;
				spec = null;
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				if (null != public_file_in) {
					public_file_in.close();
					public_file_in = null;
				}
				kfac = null;
				spec = null;
			}
	
			return public_key;
		}
	
		public static void main(String[] args){    
			RSAEncrypt rsaEncrypt= new RSAEncrypt();    

			String pubkeyPath = "keysUnreg/pub/QMLogin.pub";
			String priKeyPath = "keysUnreg/pri/QMLogin.pri";

			try {
				rsaEncrypt.readX509PublicKey(pubkeyPath);
				rsaEncrypt.readPKCS8PrivateKey(priKeyPath);

			} catch (Exception e1) {
				e1.printStackTrace();
			}
		
			//加载公钥    
			try {    


				RSAPublicKey publickey = rsaEncrypt.getPublicKey();
				System.out.println("加载公钥成功");    


				BigInteger moduls = publickey.getModulus();
				BigInteger exponent = publickey.getPublicExponent();

				BASE64Encoder encoder = new BASE64Encoder();
				String modulsStr = encoder.encode(moduls.toByteArray());
				String exponentStr = encoder.encode(exponent.toByteArray());
				System.out.println("public key moduls:");
				System.out.println(modulsStr);
				System.out.println("Public Key:");
				System.out.println(encoder.encode(publickey.getEncoded()));

			} catch (Exception e) {    
				System.err.println(e.getMessage());    
				System.err.println("加载公钥失败");    
			}    

		}
	
	}

通过以上程序得到的公私钥,是一串BASE64编码过的字符串,类似如下:

公钥内容:

MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCYD6I0/PkUGiB03ZdQrBUXhuL4vhBQB/k8DzcB
8GN8ww7kMChAv5nEKxVHLsx11Md5nGHD2gcoZCKyKobXO3+3zD+zv5g4gBNzduw8Xh9Oxlj+2Waz
Vcl20fSI1HI/RY+OlAA8B/wrSBDDU/9LCAGZVBN83RcUHruL5TXfwrnStQIDAQAB

私钥内容:

MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAJgPojT8+RQaIHTdl1CsFReG4vi+
EFAH+TwPNwHwY3zDDuQwKEC/mcQrFUcuzHXUx3mcYcPaByhkIrIqhtc7f7fMP7O/mDiAE3N27Dxe
H07GWP7ZZrNVyXbR9IjUcj9Fj46UADwH/CtIEMNT/0sIAZlUE3zdFxQeu4vlNd/CudK1AgMBAAEC
gYB/p5A6/6xyzcQ1l9lx5iUGzTw6KgUzyp3XZ8Z8IDzE/lPACRWh1bfW0XxZd9Y5jVmwpDIG40Bj
Dj16aO0uP0rlydBjxG0tEX8sNs6Z7rPJXm09FZjQHmeRuib1Bgeo5sVCF1EWfF1MssGvfpnGhXl3
Cz5TuFZj+o2/A4gcaP5cuQJBAOB2D18s7q9wWX8kkCsXSoyzh1Qb/s3v1Ye9zQHIbcFU6ZhME0TX
oLi7rqYS1mOutlafYP0iyuSwGj9NUIAngUsCQQCtbVGa2OzakMd4Suvy8R+QusCIMh60/GMzhcFR
dH2OLS8VDqbyC9QpQKcgljra1rHxo0bY2Aps1p/1VI55I3v/AkEAi3CsQfr+2FwaLQMA0NQqStgo
hNbTZwnMBASj+6yQil7+ss7n1YeC3+AwMhlXuBtMSOm/7eGUW5cO5y5XiRWBmwJBAKM/NgOmkj2i
7sCi9bs3kdjwke8iDpmawd5r129PUiiVC66sniVVUR6Lx0X7Y+c/FT05zqSrqBSPav7J21rYNg0C
QCrCGA11+l0XOziD21dHQ4I1HK1kANkGgWNt2KXWD8oL1DcR3/eJ0fEcUbMc/37WAy1Js/Alx07D
/5ch6y2xFcw=
	
###在iOS中使用RSA加密

找到了一个不错的开源库:[ios-openssl-wrapper](https://github.com/lokielse/ios-openssl-rsa-wrapper)
需要openssl-Universal库的支持,cocoapod:    

	pod 'OpenSSL-Universal', '~> 1.0.1.l'

需要BASE64支持,可选择的开源库有：GTMBase64 

加密代码：

	+(NSString*)cryptoStringWithRSA:(NSString *)plaintext
	{

		QSRSA *qr = [[QSRSA alloc] init];
		//此处公钥需要加上"BEGIN..... END..... 并且每行后加\n换行
		NSString *publicKey = @"-----BEGIN PUBLIC KEY-----\n"
		"MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCYD6I0/PkUGiB03ZdQrBUXhuL4vhBQB/k8DzcB\n"
		"8GN8ww7kMChAv5nEKxVHLsx11Md5nGHD2gcoZCKyKobXO3+3zD+zv5g4gBNzduw8Xh9Oxlj+2Waz\n"
		"Vcl20fSI1HI/RY+OlAA8B/wrSBDDU/9LCAGZVBN83RcUHruL5TXfwrnStQIDAQAB\n"
		"-----END PUBLIC KEY-----";
	
	
		NSData *plainData = [plaintext dataUsingEncoding:NSUTF8StringEncoding];
	
		//Set Public key
		[qr setPublicKey:publicKey];
	
		NSString *encryptString = [GTMBase64 stringByEncodingData:[qr encryptDataWithPublicKey:plainData]];
		return encryptString;
	}

