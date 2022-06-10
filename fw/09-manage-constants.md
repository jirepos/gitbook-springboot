# 상수 관리 

상수는 두 개의 클래스에서 관리한다. 시스템의 동작과 관련있는 상수는 `com.sogood.core.constants 패키지의 CoreConstant.java에서 관리한다. 비즈니스와  관련있는 상수는 com.sogood.biz.common.constants의 CommonConstant.java에거 관리한다.` 

## CoreConstatnts.java

이 상수 값을 담은 클래스의  상수 필드의  값은 Yaml  파일에서 읽어서 설정하도록 변경할 예정이다. 

```java
package com.sogood.core.constants;

public class CoreConstant {

  /** ThreadLocal에 저장할 HttpServletRequest의 키 */
	public static final String THREAD_LOCAL_HTTP_REQUEST_KEY = "THREAD_LOCAL_HTTP_REQUEST_KEY";
	/** ThreadLocal에 저장할 HttpServletResponse의 키 */
	public static final String THREAD_LOCAL_HTTP_RESPONSE_KEY = "THREAD_LOCAL_HTTP_RESPONSE_KEY";

	public static final String POST_IMAGE_DIR_NAME = "post/images";
	public static final String POST_DIR_NAME = "post";
	// appplication.yaml 
	/** file upload base directory */
	public static final String YAML_KEY_UPLOAD_DIR = "upload-dir"; 

}///~
```

### CommonConstant

CommonConstant는 CoreConstant 클래스를 상속받아 구현한다. 

```java
package com.sogood.biz.common.constants;

import com.sogood.core.constants.CoreConstant;

public class CommonConstant extends CoreConstant {
}
```

※ 아직 작성된 것이 없다.