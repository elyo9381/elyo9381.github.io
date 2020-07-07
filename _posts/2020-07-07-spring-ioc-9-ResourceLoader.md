---
layout: post
title: spring ResourceLoader
subtitle: "spring, framework"
categories: devlog
tags: spring
comments: true
---
> spring study

# Spring

## IoC 컨테이너 9부 : ResourceLoader

* ### ResourceLoader

  >ApplicationContext extends ​ResourceLoader
  
  resourceLoader를 통해서 리소스를 읽어올수있습니다.  

  리소스는 main폴더의 resources폴더를 뜻합니다.   
  그리고 다른곳에 위치한 resource들 또한 의미합니다. 
  리소스를 읽어오는 방법들은 
    - 파일시스템에서 읽어오기
    - 클래스패스에서 읽어오기
    - URL로 읽어오기 
    - 상대/절대 경로로 읽어오기 
  등이있습니다.
  
  ```
    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        Resource resource = resourceLoader.getResource("classpath:text.txt");
        System.out.println(resource.exists());
        System.out.println(resource.getDescription()); // 패키지 기술
        System.out.println(Files.readString(Path.of(resource.getURI())));
        System.out.println(Path.of(resource.getURI())); // string으로 된 파일경로를를 파일시스템이 인지할수있는 경로로 변환하는 메소드
        System.out.println(resource.getURI()); // 파일경로를 문자열로
        // Files.readString << java11의 메소드 파일시스템의패스를 형성해준다.
        }
    }
  ```
  위의 코드는 resourceLoader를 이용해서 리소스를 읽어오는 방법입니다.

