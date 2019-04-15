# guide-markdown-editor-build

Desktop Application Builder for guide-markdown-editor.

```yaml
jobs:
  build_desktop:
    executor:
      name: buildpack_deps
    steps:
      - checkout
      - attach_workspace:
          at: *workspace_root
      - run: |
          git config --global user.email "s+circleci@sugarshin.net"
          git config --global user.name "CircleCI"
      - run: /bin/bash .circleci/push.bash

workflows:
  test_build:
    jobs:
      - build
      - build_desktop:
          requires:
            - build
          filters:
            branches:
              only: master
```

```bash
#!/bin/bash

set -eu

readonly REPO=guide-markdown-editor-build
readonly VERSION=$(cat package.json | jq -r '.version')

[ -d $REPO ] || git clone --depth=1 git@github.com:sugarshin/$REPO.git $REPO
cd "$REPO"
rm -rf ./dist
cp -r ../dist dist
cat package.json | jq '.version = "'${VERSION}'"' > tmp
mv tmp package.json
rm -rf ./tmp
git add dist && MSG="Build ${CIRCLE_BUILD_NUM}"; git commit -m "$MSG"
git push origin master || true
```
