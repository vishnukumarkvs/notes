Terragrunt
==========

- Environments like dev, Staging should be in a folder called`live`
- Terragrunt is created for DRY - Donot Repeat Yourself
- For state management in terraform, you need to have separate bucket for each env u need to create manually and add it in backend config
- terragrunt init is optional. Ity automatically applies when you do apply

```
- live
- - terragrunt.hcl
```

- A unit in terragrunt is a directory which has terragrunt.hcl in it
- It represents a single piece of infrastructure