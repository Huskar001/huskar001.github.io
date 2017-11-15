---
layout: post
title:  "Android AES加密传输 java web 解密"
date:   2017-11-15 14:19:00 +0800
categories: android
tag: android
---
* content
{:toc}








# AES加解密

高级加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院（NIST）于2001年11月26日发布于FIPS PUB 197，并在2002年5月26日成为有效的标准。2006年，高级加密标准已然成为对称密钥加密中最流行的算法之一。

## 问题

最近一个需求,Android请求传递密码的时候，需要加密，服务端丢给一个AES加密的算法,告知密码,心想这个简单啊，调用接口而已。然后问题出来了，怎么也登录不上，查看log，调试等，发现我生成的密文，服务端解密不了，奇怪，然后客户端自己解密也挂了。好吧，这个查出来是字符编码问题，然后又发现了问题，同样的明文，客户端与服务端加密出来的密文不一样。一搜，网上有很多这样的情况。
大家说的最多的就是 Cipher cipher=Cipher.getInstance("AES"); java和android调用的默认算法不一样 需要改成一样。OK 那就改 改好了发现还是不行。胡乱找帖子看代码，就是解决不了。大部分都是一些单独的加解密。
比如要改

	SecureRandom sr = new SecureRandom();// java pc版加密设置  
	// SecureRandom sr = SecureRandom.getInstance(SHA1, CRYPTO);//
	
还有

	Cipher cipher=Cipher.getInstance("AES");改成
	Cipher cipher=Cipher.getInstance("AES/CBC/PKCS5Padding");统一化构造函数
	同时加向量参数
	cipher.init(Cipher.DECRYPT_MODE, key,new IvParameterSpec(iv));
	
能改的都改了
没办法单步调试发现

	//1.构造密钥生成器，指定为AES算法,不区分大小写
	KeyGenerator keygen=KeyGenerator.getInstance("AES");
	//2.根据ecnodeRules规则初始化密钥生成器
	//生成一个128位的随机源,根据传入的字节数组
	keygen.init(128, new SecureRandom(encodeRules.getBytes()));
	//3.产生原始对称密钥
	SecretKey original_key=keygen.generateKey();
	//4.获得原始对称密钥的字节数组
	byte [] raw=original_key.getEncoded();
	//5.根据字节数组生成AES密钥
	SecretKey key=new SecretKeySpec(raw, "AES");
	
重点就是raw不一样 ，仔细观察了下，不对，怎么引入了KeyGenerator这玩意随机数啊这是，我瞬间知道了什么情况，再仔细看看一些帖子。原来只要是隔着网络加解密的，不能用这样的写法
SecretKeySpec的第一个参数直接取密码的byte就好了 注意位数。

重点是这个问题缠了我两天。烦死了，早知道认真调试下就好了，总是相信网上的又没仔细看。不过好像也没人说起这个事。贴上代码吧


## 解决

原代码

	public class test
	{
		  public static String AESEncode(String encodeRules,String content){
			  try {
				  //1.构造密钥生成器，指定为AES算法,不区分大小写
				  KeyGenerator keygen=KeyGenerator.getInstance("AES");
				  //2.根据ecnodeRules规则初始化密钥生成器
				  //生成一个128位的随机源,根据传入的字节数组
				  keygen.init(128, new SecureRandom(encodeRules.getBytes()));
					//3.产生原始对称密钥
				  SecretKey original_key=keygen.generateKey();
					//4.获得原始对称密钥的字节数组
				  byte [] raw=original_key.getEncoded();
				  //5.根据字节数组生成AES密钥
				  SecretKey key=new SecretKeySpec(raw, "AES");
					//6.根据指定算法AES自成密码器
				  Cipher cipher=Cipher.getInstance("AES");
					//7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密解密(Decrypt_mode)操作，第二个参数为使用的KEY
				  cipher.init(Cipher.ENCRYPT_MODE, key);
				  //8.获取加密内容的字节数组(这里要设置为utf-8)不然内容中如果有中文和英文混合中文就会解密为乱码
				  byte [] byte_encode=content.getBytes("utf-8");
				  //9.根据密码器的初始化方式--加密：将数据加密
				  byte [] byte_AES=cipher.doFinal(byte_encode);
				//10.将加密后的数据转换为字符串
				  //这里用Base64Encoder中会找不到包
				  //解决办法：
				  //在项目的Build path中QueryTodoListServicrImpl先移除JRE System Library，再添加库JRE System Library，重新编译后就一切正常了。
				  String AES_encode = bytesToHexString(byte_AES);
				//11.将字符串返回
				  return AES_encode;
			  } catch (NoSuchAlgorithmException e) {
				  e.printStackTrace();
			  } catch (NoSuchPaddingException e) {
				  e.printStackTrace();
			  } catch (InvalidKeyException e) {
				  e.printStackTrace();
			  } catch (IllegalBlockSizeException e) {
				  e.printStackTrace();
			  } catch (BadPaddingException e) {
				  e.printStackTrace();
			  } catch (UnsupportedEncodingException e) {
				  e.printStackTrace();
			  }
			  
			  //如果有错就返加nulll
			  return null;         
		  }
		  /*
		   * 解密
		   * 解密过程：
		   * 1.同加密1-4步
		   * 2.将加密后的字符串反纺成byte[]数组
		   * 3.将加密内容解密
		   */
		  public static String AESDncode(String encodeRules,String content){
			  try {
				  //1.构造密钥生成器，指定为AES算法,不区分大小写
				  KeyGenerator keygen=KeyGenerator.getInstance("AES");
				  //2.根据ecnodeRules规则初始化密钥生成器
				  //生成一个128位的随机源,根据传入的字节数组
				  keygen.init(128, new SecureRandom(encodeRules.getBytes()));
					//3.产生原始对称密钥
				  SecretKey original_key=keygen.generateKey();
					//4.获得原始对称密钥的字节数组
				  byte [] raw=original_key.getEncoded();
				  //5.根据字节数组生成AES密钥
				  SecretKey key=new SecretKeySpec(raw, "AES");
					//6.根据指定算法AES自成密码器
				  Cipher cipher=Cipher.getInstance("AES");
					//7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密(Decrypt_mode)操作，第二个参数为使用的KEY
				  cipher.init(Cipher.DECRYPT_MODE, key);
				  //8.将加密并编码后的内容解码成字节数组
				  byte [] byte_content= hexToBytes(content);
				  /*
				   * 解密
				   */
				  byte [] byte_decode=cipher.doFinal(byte_content);
				  String AES_decode=new String(byte_decode,"utf-8");
				  return AES_decode;
			  } catch (NoSuchAlgorithmException e) {
				  e.printStackTrace();
			  } catch (NoSuchPaddingException e) {
				  e.printStackTrace();
			  } catch (InvalidKeyException e) {
				  e.printStackTrace();
			  } catch (IOException e) {
				  e.printStackTrace();
			  } catch (IllegalBlockSizeException e) {
				  e.printStackTrace();
			  } catch (BadPaddingException e) {
				  e.printStackTrace();
			  }
			  
			  //如果有错就返加nulll
			  return null;         
		  }
		  
		  //byte[]转String
		  public static byte[] hexToBytes(String hexStr) {
			  int len = hexStr.length();
			  hexStr = hexStr.toUpperCase();
			  byte[] des;
			  if (len % 2 != 0 || len == 0) {
			   return null;
			  } else {
			   int halfLen = len / 2;
			   des = new byte[halfLen];
			   char[] tempChars = hexStr.toCharArray();
			   for (int i = 0; i < halfLen; ++i) {
				char c1 = tempChars[i * 2];
				char c2 = tempChars[i * 2 + 1];
				int tempI = 0;
				if (c1 >= '0' && c1 <= '9') {
				 tempI += ((c1 - '0') << 4);
				} else if (c1 >= 'A' && c1 <= 'F') {
				 tempI += (c1 - 'A' + 10) << 4;
				} else {
				 return null;
				}
				if (c2 >= '0' && c2 <= '9') {
				 tempI += (c2 - '0');
				} else if (c2 >= 'A' && c2 <= 'F') {
				 tempI += (c2 - 'A' + 10);
				} else {
				 return null;
				}
				des[i] = (byte) tempI;
				// System.out.println(des[i]);
			   }
			   return des;
			  }
			 }
			   
		  //String转byte[]
		  public static String bytesToHexString(byte[] src){
			  StringBuilder stringBuilder = new StringBuilder("");
			  if (src == null || src.length <= 0) {
				  return null;
			  }
			  for (int i = 0; i < src.length; i++) {
				  int v = src[i] & 0xFF;
				  String hv = Integer.toHexString(v);
				  if (hv.length() < 2) {
					  stringBuilder.append(0);
				  }
				  stringBuilder.append(hv);
			  }
			  return stringBuilder.toString();
		  }
		  
		  public static void main(String[] args) {
			  String pwd = "1qaz";
			  String encodeRules = "1111111111111111";
			  System.out.println("加密前：   "+pwd);
			  String jiami = AESEncode(encodeRules, pwd);
			  System.out.println("加密后：   "+jiami);
			  String jiemi = AESDncode(encodeRules, jiami);
			  System.out.println("解密后：   "+jiemi);
		  }

	}
	
	
改后

	public class AESCrypt2 {
		private static byte[] iv = { 1, 2, 3, 4, 5, 6, 7, 8,9,1,1,1,2,3,4,5};
		public static String AESEncode(String encodeRules,String content){
			try {
				SecretKey key=new SecretKeySpec(encodeRules.getBytes("utf-8"), "AES");
				//6.根据指定算法AES自成密码器
				Cipher cipher=Cipher.getInstance("AES/CBC/PKCS5Padding");
				//7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密解密(Decrypt_mode)操作，第二个参数为使用的KEY
				cipher.init(Cipher.ENCRYPT_MODE, key,new IvParameterSpec(iv));
				//8.获取加密内容的字节数组(这里要设置为utf-8)不然内容中如果有中文和英文混合中文就会解密为乱码
				byte [] byte_encode=content.getBytes("utf-8");
				//9.根据密码器的初始化方式--加密：将数据加密
				byte [] byte_AES=cipher.doFinal(byte_encode);
				//10.将加密后的数据转换为字符串
				//这里用Base64Encoder中会找不到包
				//解决办法：
				//在项目的Build path中QueryTodoListServicrImpl先移除JRE System Library，再添加库JRE System Library，重新编译后就一切正常了。
				String AES_encode = bytesToHexString(byte_AES);
				//String AES_encode = Base64.encode(byte_AES);
				//11.将字符串返回
				return AES_encode;
			} catch (NoSuchAlgorithmException e) {
				e.printStackTrace();
			} catch (NoSuchPaddingException e) {
				e.printStackTrace();
			} catch (InvalidKeyException e) {
				e.printStackTrace();
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			} catch (InvalidAlgorithmParameterException e) {
				e.printStackTrace();
			}

			//如果有错就返加nulll
			return null;
		}
		/*
		 * 解密
		 * 解密过程：
		 * 1.同加密1-4步
		 * 2.将加密后的字符串反纺成byte[]数组
		 * 3.将加密内容解密
		 */
		public static String AESDncode(String encodeRules,String content){
			try {
				SecretKey key=new SecretKeySpec(encodeRules.getBytes(), "AES");
				//6.根据指定算法AES自成密码器
				Cipher cipher=Cipher.getInstance("AES/CBC/PKCS5Padding");
				//7.初始化密码器，第一个参数为加密(Encrypt_mode)或者解密(Decrypt_mode)操作，第二个参数为使用的KEY
				cipher.init(Cipher.DECRYPT_MODE, key,new IvParameterSpec(iv));
				//8.将加密并编码后的内容解码成字节数组
				byte [] byte_content= hexToBytes(content);
				//byte [] byte_content= Base64.decode(content);
				  /*
				   * 解密
				   */
				byte [] byte_decode=cipher.doFinal(byte_content);
				String AES_decode=new String(byte_decode,"utf-8");
				return AES_decode;
			} catch (NoSuchAlgorithmException e) {
				e.printStackTrace();
			} catch (NoSuchPaddingException e) {
				e.printStackTrace();
			} catch (InvalidKeyException e) {
				e.printStackTrace();
			} catch (IOException e) {
				e.printStackTrace();
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			} catch (InvalidAlgorithmParameterException e) {
				e.printStackTrace();
			}

			//如果有错就返加nulll
			return null;
		}

		//byte[]转String
		public static byte[] hexToBytes(String hexStr) {
			int len = hexStr.length();
			hexStr = hexStr.toUpperCase();
			byte[] des;
			if (len % 2 != 0 || len == 0) {
				return null;
			} else {
				int halfLen = len / 2;
				des = new byte[halfLen];
				char[] tempChars = hexStr.toCharArray();
				for (int i = 0; i < halfLen; ++i) {
					char c1 = tempChars[i * 2];
					char c2 = tempChars[i * 2 + 1];
					int tempI = 0;
					if (c1 >= '0' && c1 <= '9') {
						tempI += ((c1 - '0') << 4);
					} else if (c1 >= 'A' && c1 <= 'F') {
						tempI += (c1 - 'A' + 10) << 4;
					} else {
						return null;
					}
					if (c2 >= '0' && c2 <= '9') {
						tempI += (c2 - '0');
					} else if (c2 >= 'A' && c2 <= 'F') {
						tempI += (c2 - 'A' + 10);
					} else {
						return null;
					}
					des[i] = (byte) tempI;
					// System.out.println(des[i]);
				}
				return des;
			}
		}

		//String转byte[]
		public static String bytesToHexString(byte[] src){
			StringBuilder stringBuilder = new StringBuilder("");
			if (src == null || src.length <= 0) {
				return null;
			}
			for (int i = 0; i < src.length; i++) {
				int v = src[i] & 0xFF;
				String hv = Integer.toHexString(v);
				if (hv.length() < 2) {
					stringBuilder.append(0);
				}
				stringBuilder.append(hv);
			}
			return stringBuilder.toString();
		}
	}

实测 在android中加密然后传输到web java去解密是可行的。知其然更要知其所以然啊。上学没好好听的课，回头还要补回来的。
