
## 1 ##
- fzaninotto/faker[v1.9.0, ..., v1.9.2] require php ^5.3.3 || ^7.0 -> your php version (8.0.6) does not satisfy that requirement.

solve:

change on composer.json

 "fzaninotto/faker": "^1.9"
to
 "fzaninotto/faker": "^1.9.x-dev"



