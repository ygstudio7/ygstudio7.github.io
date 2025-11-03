`git`에서 여러 개의 커밋을 합치는 방법은 **squash** 또는 **rebase**를 사용하는 것입니다. 아래는 구체적인 단계별 방법입니다.


## **방법 1: Merge Commit**
합치고자 하는 커밋들이 다른 브랜치에 있을 때는, 병합 커밋 (Merge Commit)을 사용할 수 있습니다:
```bash
git merge --squash <branch_name>
git commit -m "Combined commit from branch"
```


---

## **방법 2: Git Interactive Rebase (`git rebase -i`)**
합치고자 하는 커밋이 현재 브랜치에 있을때는 **rebase** 와 **squash** 을 사용할 수 있습니다.

1. **기준 커밋 선택**  
   변경하려는 커밋 범위에서 **기준 커밋**을 선택합니다. 예를 들어, 마지막 3개의 커밋을 합치려면:
   ```bash
   git rebase -i HEAD~3
   ```

2. **Interactive Rebase 편집**  
   실행하면 다음과 같은 편집 화면이 나타납니다:
   ```plaintext
   pick 123abc First commit
   pick 456def Second commit
   pick 789ghi Third commit
   ```

   - 첫 번째 커밋(`pick 123abc`)은 그대로 두고, 나머지 커밋을 **squash**로 변경합니다:
     ```plaintext
     pick 123abc First commit
     squash 456def Second commit
     squash 789ghi Third commit
     ```

3. **커밋 메시지 수정**  
   squash가 설정된 커밋들의 메시지를 합치거나 수정하는 화면이 나타납니다:
   ```plaintext
   # This is a combination of 3 commits.
   # The first commit's message is:
   First commit
   
   # The following commit messages will also be included:
   Second commit
   Third commit
   ```
   - 원하는 메시지로 변경 후 저장하고 종료.

4. **Rebase 완료**  
   Git이 커밋을 합친 후 적용합니다. 충돌이 발생하면 수정 후 아래 명령을 실행하세요:
   ```bash
   git rebase --continue
   ```

---

## **주의사항**

- 충돌이 발생하면 적절히 해결하고 `git rebase --continue`를 실행하세요.

---

