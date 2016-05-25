---
layout: post
title: Android应用签名通过Dialog填写
category: 移动开发
tags: android
keywords: android,应用签名
---

通过build.gradle创建项目发布时填写应用签名的弹窗

Asking android release details throught build.gradle

	import groovy.swing.SwingBuilder

	gradle.taskGraph.whenReady { taskGraph ->
	    if(taskGraph.hasTask(':app:assembleRelease')) {
	        def storePass = ''
	        def keyPass = ''
	        def keyAlias = ''
	        if(System.console() == null) {
	            new SwingBuilder().edt {
	                dialog(modal: true, title: 'Enter password', alwaysOnTop: true, resizable: false, locationRelativeTo: null, pack: true, show: true) {
	                    vbox {
	                        // Put everything below each other
	                        label(text: "Please enter store passphrase:")
	                        def input1 = passwordField()
	                        label(text: "Please enter key passphrase:")
	                        def input2 = passwordField()
	                        label(text: "Please enter key alias:")
	                        def input3 = readLine()
	                        button(defaultButton: true, text: 'OK', actionPerformed: {
	                            storePass = input1.password;
	                            keyPass = input2.password;
	                            keyAlias = input3.password;
	                            dispose();
	                        })
	                    }
	                }
	            }
	        } else {
	            storePass = System.console().readPassword("\nPlease enter store passphrase: ")
	            keyPass = System.console().readPassword("Please enter key passphrase: ")
	            keyAlias = System.console().readPassword("Please enter key alias: ")
	        }
	        if(storePass.size() <= 0 || keyPass.size() <= 0 || keyAlias.size() <= 0) {
	            throw new InvalidUserDataException("You must enter the passwords to proceed.")
	        }
	        storePass = new String(storePass)
	        keyPass = new String(keyPass)
	        keyAlias = new String(keyAlias)
	        android.signingConfigs.release.storePassword = storePass
	        android.signingConfigs.release.keyPassword = keyPass
	        android.signingConfigs.release.keyAlias = keyAlias
	    }
	}