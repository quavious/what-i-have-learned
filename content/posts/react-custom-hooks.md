+++
title = "React Custom Hooks - useInput & useFileReader"
date = "2021-04-01T00:12:53+09:00"
author = ""
authorTwitter = "" #do not include @
cover = "Neon.Owl"
tags = ["React", "Hooks", "State", "Effect", "Component", "Typescript", "NodeJS"]
keywords = ["React", "Hooks", "State", "Effect", "Component", "Typescript", "NodeJS"]
description = ""
showFullContent = false
+++

# 리액트 커스텀 훅스 만들어서 사용하기

## useInput

input 태그의 value와 onChange 함수를 반복 사용하는 것을 막기 위해 만들었습니다.

```TSX
export type validator = ((e: string) => boolean) | null;
export const useInput = (validator: validator = null) => {
  const [value, setValue] = useState("");
  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const tmp = e.target.value;
    if (!validator) {
      setValue(tmp);
    } else if (validator(tmp[tmp.length - 1])) {
      setValue(tmp);
    } else {
      setValue(value);
    }
  };
  const clearValue = () => setValue("")
  return {
    value,
    onChange,
    clearValue,
  };
};
export const App = () => {
	const validator = (e: string) => e !== " ";
  const {clearValue as clearEmail, ...inputEmail} = useInput();
	return (
		<div>
			<input type="text" {...inputEmail}>
			<button onClick={clearValue}>Submit!</button>
		</div>
	);
}
```

useInput은 입력값을 검증하는 validator 함수를 선택적으로 받아서 value, onChange, clearValue를 반환하는 Hooks이다.

validator가 설정되어있으면 validator 함수가 true를 반환할 때에만 상태값이 변한다.

value, onChange는 jsx, tsx에서 기본적으로 제공되는 기능이므로 ES6 비구조화할당으로 넣어줘도 알아서 잘 작동한다.

```jsx
const [value, setValue] = useState("")
<input value={value} onChange={onChange}/>

const inputValue = useState("")
<input {...inputValue}/>
```

버튼을 클릭한 뒤 자동으로 상태값이 초기화되도록 clearValue 함수를 따로 작성했다.

## useFileReader

NextJS는 서버 사이드 렌더링을 지원한다.

서버 사이드 렌더링은 간단히 말하면 서버에서 페이지를 생성한 뒤 클라이언트, 웹 브라우저에 전달해서 렌더링하도록 하는 방식이다.

NextJS를 사용하면 검색 엔진이 웹 페이지를 더 잘 인식할 수 있다. 따라서 검색 엔진 최적화에 도움이 된다.

그리고 웹 브라우저의 부담도 감소시킬 수 있다.

하지만 웹 브라우저에서만 사용할 수 있는 몇몇 Web API들이 사용이 제한되는 경우가 있다.

파일을 불러오는 FileReader나, 양식 데이터를 생성하는 FormData를 예로 들 수 있다.

useEffect는 웹 페이지가 웹 브라우저에 렌더링되었을 때 실행되므로, 이 점을 활용하면 Web API도 무리없이 사용할 수 있다.

FileReader를 예시로 들었다.

```TSX
export const useFileReader = () => {
	const [reader, setReader] = useState<FileReader | null>(null)
	const [isReady, setReady] = useState(false)
	useEffect(() => {
		const tmp = new FileReader();
		setReader(tmp);
		setReady(true)
	}, [])

	return {reader, isReady};
}

export const App = () => {
	const {reader, isReady} = useFileReader();
	const handleChange = (e) => {
		e.preventDefault();
		if(!isReady) return;
		const image = e.target.files[0];
    reader.onloadend = (e) => {
      console.log(e.target.result)
    };
    reader.readAsDataURL(image);
	}
	return (
		<div>
			<form encType="multipart/form-data">
				<input
					type="file"
					id="image"
					accept="image/*"
					onChange={handleChange}
				/>
			</form>
		</div>
	)
}
```

FileReader 객체를 담을 reader 상태와 reader가 생성되었는지 판단하는 isReady 상태를 생성했다.

useFileReader가 한번 실행되면, 함수 내부의 useEffect가 작동하여 setReader로 reader와 isReady 상태를 업데이트한다.

이후 리턴한 객체는 컴포넌트에서 받아서 사용할 수 있다.

input 태그에 이미지 파일이 입력되면, handleChange 함수가 작동한다.

이 때 reader 객체가 아직 준비가 안 되었으면, 즉 isReady 값이 false라면 함수는 그대로 리턴한다.

isReady 값이 true라면 파일을 읽어들인다.

파일을 읽는데 성공했다면, 콘솔 창에 결과를 확인할 수 있다.
