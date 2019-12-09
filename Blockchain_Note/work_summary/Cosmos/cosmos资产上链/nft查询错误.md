After I mint an nft,  



I am getting the following error while I executed the following query command,

:gaiacli query nft supply crypto-kitties `

> ERROR: cannot parse disfix JSON wrapper: invalid character '\x01' looking for beginning of value

The query method for `cliCtx.QueryWithData` has no errors and the `res= [1 0 0 0 0 0 0 0]`.

```go
// GetCmdQueryCollectionSupply queries the supply of a nft collection
func GetCmdQueryCollectionSupply(queryRoute string, cdc *codec.Codec) *cobra.Command {
 
         res, _, err := cliCtx.QueryWithData(fmt.Sprintf("custom/%s/supply/%s", queryRoute, denom), bz)
         if err != nil {
            return err
         }

         var out exported.NFT
         err = cdc.UnmarshalJSON(res, &out)
         if err != nil {
            return err
         }

         return cliCtx.PrintOutput(out)
      },
   }
}
```

> This method `err = cdc.UnmarshalJSON(res, &out)` gives an error:
>
> > ERROR: cannot parse disfix JSON wrapper: invalid character '\x01' looking for beginning of value