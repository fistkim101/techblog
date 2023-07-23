# 프로토타입 패턴

## 대분류

객체 생성



## 문제상황

생성 비용이 크다는 전제 하에 똑같은 해당 객체가 또 필요하다던가 동일 타입의 객체지만 프로퍼티가 약간 다른 (결국 동등하지 않은 객체) 객체를 또 만들어야 하는 상황인 경우이다.

생성 비용이 크다는 말은 해당 객체의 생성을 위해서 거쳐야할 로직이 많다던가 외부 API 와의 통신이 필요하다던가 등 여러 의미가 있을 수 있다.

다시 정리하자면 생성 비용이 큰 객체인데 동일하거나 비슷한 객체가 또 필요한 경우에 프로토타입 패턴을 사용할 수 있다.



## 해결방안

생성 비용이 큰 상황이기 때문에 기존에 있는 객체를 복제하여 커스텀 해서 사용한다.



## 얕은 복사, 깊은 복사

프로토타입 패턴은 기존 객체를 '복제'한다는 것이 핵심이고, 이때 알아야 할 개념이 얕은 복사와 깊은 복사이다.

> 얕은 복사(Shallow Copy)와 깊은 복사(Deep Copy)는 데이터를 복사하는 방식에 대한 개념입니다. 이 둘은 복사되는 데이터의 구조와 참조에 따라 다르게 동작합니다.
>
> 1. 얕은 복사 (Shallow Copy): 얕은 복사는 원본 객체의 주소를 공유하여, 복사된 객체는 원본 객체의 참조를 가리키는 방식입니다. 즉, 객체의 필드가 참조 타입인 경우 해당 필드는 동일한 주소를 가리키게 됩니다. 따라서 한 객체의 변경이 다른 객체에 영향을 미칠 수 있습니다.
>
> 얕은 복사는 복사 과정이 빠르고 메모리 사용이 감소하는 장점이 있지만, 참조된 객체의 내용이 변경되면 원본 객체와 복사된 객체 모두에 영향을 미칠 수 있으므로 주의가 필요합니다.
>
> 2. 깊은 복사 (Deep Copy): 깊은 복사는 원본 객체의 내용을 완전히 새로운 메모리에 복사하여, 복사된 객체와 원본 객체는 서로 독립적인 데이터를 가지게 됩니다. 즉, 객체의 모든 필드와 참조된 객체들까지 모두 새로운 객체로 복사됩니다.
>
> 깊은 복사는 객체의 모든 내용을 복사하므로 원본 객체의 변경이 복사된 객체에 영향을 미치지 않습니다. 하지만 복사 과정이 복잡하고 시간이 오래 걸릴 수 있으며, 메모리 사용이 증가하는 단점이 있습니다.
>
> 자바에서는 얕은 복사를 쉽게 수행할 수 있는 방법이 제공되지만, 깊은 복사는 사용자가 직접 구현해야 합니다. 객체를 복사할 때 얕은 복사와 깊은 복사 중 적절한 방법을 선택하여 사용해야 합니다. 만약 객체의 구조가 간단하고 참조된 객체의 변경이 원본 객체에 영향을 미치는 것이 용인되는 경우에는 얕은 복사를 사용할 수 있습니다. 그러나 객체의 구조가 복잡하거나 완전한 독립성이 필요한 경우에는 깊은 복사를 구현하여 사용해야 합니다.

그래서 얕은 복사시에 내부 프로퍼티가 동일한 주소값을 지니게 된다. 위 설명에서 '얕은 복사는 원본 객체의 주소를 공유하여' 라는 의미는 할당되는 변수의 주소값이 같다는 이야기가 아니고 각 변수의 주소값은 다르지만 해당 객체 내부의 프로퍼티가 참조형일 경우 주소를 공유한다는 의미이다.(후속 설명인 '객체의 필드가 참조 타입인 경우 해당 필드는 동일한 주소를 가리킨다'는 의미와 동일)

그래서 객체의 프로퍼티가 모두 원시형인 경우에&#x20;



## 실습코드

```java
@Getter
@Setter
public class GitHubIssue implements Cloneable, Serializable {

    private String name;
    private String type;
    private String description;

    private GitHubRepository gitHubRepository;

    public GitHubIssue(String name, String type, String description, GitHubRepository gitHubRepository) {
        this.name = name;
        this.type = type;
        this.description = description;
        this.gitHubRepository = gitHubRepository;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

```

```java
@Getter
@Setter
public class GitHubRepository implements Serializable {

    private String url;

    public GitHubRepository(String url) {
        this.url = url;
    }
}
```

```java
public class DeepCopySupport {

    @SuppressWarnings("unchecked")
    public static <T> T deepCopy(T original) throws IOException, ClassNotFoundException, IOException {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(original);

        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return (T) ois.readObject();
    }

}
```

```java
public class Client {
    public static void main(String[] args) throws CloneNotSupportedException, IOException, ClassNotFoundException {
        GitHubRepository gitHubRepository = new GitHubRepository("repository url 1");
        GitHubIssue original = new GitHubIssue("name1", "bug", "description1", gitHubRepository);
        GitHubIssue shallowCopied = (GitHubIssue) original.clone();
        GitHubIssue deepCopied = DeepCopySupport.deepCopy(original);

        // 시사점 1 : 얕은 복사나 깊은 복사 모두 복사본이 가리키는 주소값은 원본과 다르다
        System.out.println(original == shallowCopied); // false
        System.out.println(original == deepCopied); // false

        // 시사점 2 : 객체의 프로퍼티가 참조형인 경우 얕은 복사는 같은 주소값을 가리키며, 깊은 복사의 경우 다른 주소값을 가리킨다(완전히 새로 만들었기 때문)
        System.out.println(System.identityHashCode(original.getName()) == System.identityHashCode(shallowCopied.getName())); // true
        System.out.println(System.identityHashCode(original.getName()) == System.identityHashCode(deepCopied.getName())); // false
        System.out.println(original.getGitHubRepository() == shallowCopied.getGitHubRepository()); // true
        System.out.println(original.getGitHubRepository() == deepCopied.getGitHubRepository()); // false

        // 시사점 3 : 객체의 프로퍼티 중 String 의 경우 참조형이므로 같은 주소값을 가지지만 새로 할당해주면 새로운 주소값을 가리키게 되기 때문에 원본 값 변경시 복사본에 영향이 없다.
        original.setName("changed name");
        System.out.println(shallowCopied.getName().equals("changed name")); // false (set 하면서 완전히 다른 주소값의 객체가 original 에 할당되었기 때문에 복제본에 영향 없음)

        original.getGitHubRepository().setUrl("changed url");
        System.out.println(!deepCopied.getGitHubRepository().getUrl().equals("changed url")); // true (딥카피 했기에 원래 GitHubRepository 자체가 원본과 복제본이 달랐던 것이라 영향이 없음)
    }
}
```

```java
    /**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     *
     * @implSpec
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown. Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code clone} method of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return     a clone of this instance.
     * @throws  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
     * @see java.lang.Cloneable
     */
    @IntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
```

자바에서 제공해주는 Clonable 을 이용해서 기본적으로 "shallow copy"가 가능하다. 주요 정리 사항은 Client 코드 내 주석으로 정리해놨는데 조금 더 요약을 하자면 아래와 같다.

1. 얕은 복사시 참조형 프로퍼티는 동일한 주소값을 지니게 된다. 그래서 원본이든 복사본이든 동일한 주소값을 공유하고 있는 참조형 프로퍼티를 변경하게 되면 이를 공유하고 있는 원본 또는 복사본 서로에게 영향을 주게 된다.
2. 하지만 String 의 경우 동일한 참조형 프로퍼티를 가지고 있지만 새 값을 할당해주면 주소값이 변경되기 때문에 원본 또는 복사본 서로에게 영향을 주는 일이 없다.

deep copy 는 행위 자체가 shallow copy 에 비해 cost 가 높다는 것을 고려해야하지만 엄청 빈도 높게 호출되지 않는다면 shallow copy 에 비해서 훨씬 안전하니까 deep copy 를 가급적 사용하도록 하는 것도 괜찮은 판단인 것 같다.
