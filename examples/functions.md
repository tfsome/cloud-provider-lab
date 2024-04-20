# Functions

The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values.

### Syntax
```
<FUNCTION NAME>(<ARGUMENT 1>, <ARGUMENT 2>, ...)
```

The function name specifies which function to call. Each defined function expects a specific number of arguments with specific value types, and returns a specific value type as a result.<p>
Some functions take an arbitrary number of arguments. For example, the min function takes any amount of number arguments and returns the one that is numerically smallest:

## Examples

Using ```terraform console``` it is possible to test various terraform functions
```
# terraform console
>
```
### Using function ```cidrsubnets``` for spliting CIDR on smaller CIDR ranges for the future subnets
```
> cidrsubnets("10.1.0.0/16", 4, 4, 8, 4)
tolist([
  "10.1.0.0/20",
  "10.1.16.0/20",
  "10.1.32.0/24",
  "10.1.48.0/20",
])
```

### Getting the hash of the file
```
> base64sha256(file("main.py"))
"GKhaoHPbLizWLKDf/MFrViFzXLcwint48gtML301z6w="
```

### Getting the current timestamp
```
> timestamp()
"2022-10-19T08:45:49Z"
```

## References
[Terraform documentation](https://www.terraform.io/cdktf/concepts/functions)
