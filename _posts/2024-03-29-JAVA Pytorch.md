---
title: "JAVA Pytorch"
last_modified_at: 2024-03-29
author: 최선빈
---

본 포스팅은 JAVA Pytorch 결과에 대한 내용 입니다.

---
&nbsp;

> ## Pytorch for JAVA
   
   * Pytorch는 페이스북에서 개발한 오픈소스 머신 러닝 프레임워크로, 딥 러닝 및 다양한 머신 러닝 작업을 수행 할 수 있습니다.
   * Pytorch JAVA를 사용하면 JAVA에서도 Pytorch 라이브러리를 사용할 수 있습니다.

         Pytorch JAVA 라이브러리를 의존성에 추가합니다.
         <!-- pom.xml -->
         <dependency>
     	 <groupId>com.facebook.soloader</groupId>
     	 <artifactId>nativeloader</artifactId>
     	 <version>0.10.1</version>
     	 <scope>runtime</scope>
          </dependency>
          <dependency>
             <groupId>com.facebook.fbjni</groupId>
             <artifactId>fbjni-java-only</artifactId>
             <version>0.2.2</version>
             <scope>runtime</scope>
          </dependency>
    
          <dependency>
             <groupId>org.pytorch</groupId>
             <artifactId>pytorch_java_only</artifactId>
             <version>1.12.1</version>
          </dependency>
    
          <dependency>
             <groupId>ai.djl.pytorch</groupId>
             <artifactId>pytorch-jni</artifactId>
             <version>2.0.0-0.22.1</version>
          </dependency>


* 이후 머신 러닝 모델 파일(.pt 형식)을 resources 폴더에 추가합니다.
  * Pytorch 모델을 불러옵니다.

           import org.pytorch.IValue;
           import org.pytorch.Module;
           import org.pytorch.Tensor;
        
           // ...
        
           public class PyTorchService {
           private Module module;

           public PyTorchService() {
           // 모델 파일을 불러와 모듈을 생성합니다.
                try (InputStream is = getClass().getResourceAsStream("/model.pt")) {
                module = Module.load(is);
                } catch (IOException e) {
                    throw new RuntimeException("모델 파일을 불러올 수 없습니다.", e);
                }
           }

          // 입력 데이터를 사용하여 머신 러닝 모델을 실행하고 결과를 반환합니다.
          public float[] predict(float[] inputData) {
              Tensor input = Tensor.fromBlob(inputData, new long[]{1, inputData.length});
              Tensor output = module.forward(IValue.from(input)).toTensor();
              float[] result = output.getDataAsFloatArray();
              return result;
            }
          }

* 위 구조를 활용하면 머신 러닝 모델을 실행하고 결과를 얻을 수 있습니다.

&nbsp;
----

> 참고 문서

[1] [블로그](https://gdngy.tistory.com/114)

