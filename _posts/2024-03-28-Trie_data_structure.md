---
title: "Trie 자료 구조 구현하기"
last_modified_at: 2024-03-28
author: 이현지
---

본 포스팅은 Trie 자료 구조를 구현하는 방법에 대한 글입니다.

<br><br>

# Trie 자료구조
Retrieval Tree(탐색 트리)의 약자<br>
문자열을 저장하고 효율적으로 탐색하기 위한 트리 기반의 자료 구조<br>
<br>
# 장점 및 활용
- 문자열 검색 시간 단축<br>
for문의 경우 특정 문자열을 찾기 위해 전체 문자열을 순회해야 함<br>
Trie 구조는 문자열 길이만큼 노드를 검사하면 됨<br>

- 자동 완성 검색과 사전 탐색에 유용함<br>
<br>
# 단점
- 메모리 측면에서 비효율적<br>
각 노드가 자식 노드들을 저장하고 있음<br>
<br>
# 구조 (트리 구조와 동일)
![trie 구조 예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcc8fGF%2FbtsGa9g2UAF%2FlNMIphrx021IbKsdUA4E91%2Fimg.png)

<br>

# 구현

Map<Character, Node><br> 
- key : 문자<br>
- value : 자식 노드 객체<br>
<br>

![trie 구현 예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F8K5eX%2FbtsF8xcCsHL%2F9ka2y3Vi755T1OtU3iizW1%2Fimg.png)


<br>

# 소스 코드
```java
public class Trie {

	@Data
	private class Node {
				
		/** 자식 노드 목록 맵 */
		private Map<Character, Node> childNodeMap = new HashMap<>();
        /** 터미널 노드 여부 */
        private boolean terminal;
        
	}

	
	/** 루트 노드 */
	private Node rootNode = new Node();	


	/**
	 * Trie에 문자열 저장
	 * 
	 * @param str 
	 * @throws Exception
	 */
	public void insert(String str) throws Exception {
				
		// 현재 노드 (for문에서 다음 문자로 넘어갈 때마다 현재 노드의 자식 노드로 넘어감)
		Node currNode = this.rootNode;

		// 각 문자마다 현재 노드의 자식 노드에 있는지 검사
	    for(int charIndex = 0; charIndex < str.length(); charIndex++) {
	    	
	    	// 현재 문자
	    	char currentChar = str.charAt(charIndex);
	    	
	    	// 현재 문자가 자식 노드에 있으면 해당 자식 노드를 현재 노드로 저장 
	    	// 아니면 현재 문자를 자식 노드로 새로 생성하고, 해당 노드를 현재 노드로 저장
	    	// computeIfAbsent() : Map에 특정키에 해당하는 값이 존재하는지 확인 후 있으면 해당 값 반환, 없으면 새로 만들어서 추가 후 반환
	    	currNode = currNode.childNodeMap.computeIfAbsent(currentChar, key -> new Node());
	    		    
	    }
		
        // 마지막 문자 표시
        currNode.terminal = true;
        
	}

	
	/**
	 * Trie에서 문자열 검색
	 * 
	 * @param target 검색할 문자열
	 * @return 검색 결과가 있을 경우 true, 없을 경우 false
	 * @throws Exception
	 */
	public boolean search(String target) throws Exception {
		//TODO 기준 문자열 받아서 목록 문자열과 비교

		// 현재 노드 (for문에서 다음 문자로 넘어갈 때마다 현재 노드의 자식 노드로 넘어감)
		Node currNode = this.rootNode;

		// 문자열의 각 문자마다 노드가 존재하는지 체크
		for(int charIndex = 0; charIndex < target.length(); charIndex++) {
			
			// 현재 문자
			char currChar = target.charAt(charIndex);
			// 현재 문자가 현재 노드의 자식 노드로 존재하면 해당 자식 노드를 현재 노드로 저장
            // 아니면 null
			currNode = currNode.childNodeMap.getOrDefault(currChar, null);
			
			if(currNode == null) {
				
				// 현재 노드가 null이면 일치하는 문자열 없음
				return false;
	        
			}
		
		}
		
		// target 문자열(예. ABC)의 마지막 문자(C)까지 일치하고, 
        // 현재 노드가 마지막 노드인지도 확인해야 함 (ABC != ABCD)
		return currNode.terminal;

	}
	
	
}
```
<br>

# 저장 및 검색 테스트
```java
public static void main(String[] args) throws Exception {

    Trie trie = new Trie(); 

    // 문자열 저장
    trie.insert("car");
    trie.insert("cake");
    trie.insert("cute");
    trie.insert("key");
    trie.insert("keep");

    // 문자열 검색
    System.out.println(trie.search("car"));
    System.out.println(trie.search("carr"));
    System.out.println(trie.search("cake"));
    System.out.println(trie.search("caker"));
    System.out.println(trie.search("cute"));
    System.out.println(trie.search("cutie"));
    System.out.println(trie.search("key"));
    System.out.println(trie.search("kenya"));
    System.out.println(trie.search("keep"));
    System.out.println(trie.search("kep"));

}
```
<br>
이상으로 Trie 자료 구조를 구현하는 방법에 대해 알아보았습니다. 

<br><br>
 > Reference 
- https://velog.io/@kimdukbae/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%ED%8A%B8%EB%9D%BC%EC%9D%B4-Trie<br>
- https://codingnojam.tistory.com/40<br>