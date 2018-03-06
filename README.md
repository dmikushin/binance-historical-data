```
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠟⠁⠀⠉⠻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⠟⠁⠀⠀⣀⠀⠀⠈⠻⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣅⠀⠀⣠⣾⣿⣷⣄⠀⢀⣼⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⠟⠉⠻⢿⣷⣾⡿⠛⠉⠻⣿⣷⣿⡿⠋⠈⠻⣿⣿⣿⣿
⣿⣿⣿⣿⣦⣀⣴⣿⡿⢿⣿⣦⣀⣴⣿⡿⢿⣷⣦⣀⣴⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⡋⠀⠀⠙⢿⣿⡿⠋⠀⠀⢹⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣦⡀⠀⠀⠉⠀⠀⣀⣴⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣦⣀⠀⣠⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
```

## Binance Bincoin Exchange historical trading data

This repository present the data for all Binance symbols from the beginning of the trading till **the end of February 2018**. 

### Data format

Trades for individual symbols are subdivided into files. Each file is compressed with TAR BZ2 algorithm and must be first decompressed:

```
$ tar -xjf SYMBOL.dat.tar.bz2
```

Trades are stored in binary format, sorted by trade id in ascending order:

```
struct Trade
{
	double price;
	double qty;
	long id, time;
	bool isBestMatch;
	bool isBuyerMaker;
};
```
Library for accessing Binance Bincoin Exchange using web sockets and JSON.

### Reading the trade data

The following code snippet demonstrates how to read the sequence of trades from the data file:

```
ifstream history("SYMBOL.dat", ifstream::binary);
if (history.is_open())
{
	history.seekg(0, history.end);
	size_t length = history.tellg();
	
	if (length % sizeof(Trade))
	{
		fprintf(stderr, "File length %zu is not a multiply of the trade record size %zu\n",
			length, sizeof(Trade));
		fprintf(stderr, "malformed history data file or invalid format?\n");
		exit(-1);
	}

	history.seekg(0, history.beg);

	const size_t szbatch = 1024;
	for (size_t j = 0, je = length / sizeof(Trade); j < je; j += szbatch)
	{
		vector<Trade> trades(szbatch);
		history.read((char*)&trades[0], sizeof(Trade) * min(szbatch, je - j));
		if (history.rdstate())
		{
			fprintf(stderr, "Error reading historical data file: ");
			if ((history.rdstate() & ifstream::eofbit) != 0)
				fprintf(stderr, "End-of-File reached on input operation\n");
			else if ((history.rdstate() & ifstream::failbit) != 0)
				fprintf(stderr, "Logical error on i/o operation\n");
			else if ((history.rdstate() & ifstream::badbit) != 0)
				fprintf(stderr, "Read/writing error on i/o operation\n");
			exit(1);
		}
	
		for (int k = 0, ke = min(szbatch, je - j); k < ke; k++)
		{
			const Trade& trade = trades[k];
		
			// TODO Here do whatever you need with the individual trade
		}
	}

	history.close();
}
```

### Further info

This data has been collected through the [Binance Public API](https://github.com/binance-exchange/binance-official-api-docs/blob/master/rest-api.md) using the [bihistorian](https://github.com/dmikushin/bitrader) tool.

###Liability

Use this program at your own risk. None of the contributors to this project are liable for any loses you may incur. Be wise and always do your own research.
