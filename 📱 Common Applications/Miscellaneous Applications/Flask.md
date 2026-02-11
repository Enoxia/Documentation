There are a lots of exploitation on [Flask](https://flask.palletsprojects.com/en/stable/) available (look for CVEs if we can retrieve the version in use).

# Console Debug PIN Exploit
In coming... Look at the CPTS Report or Google :)

Need LFI or a way to read some local files !

# Flask Unsign
We can attemp to uncover a Flask server's secret key by taking a signed session verifying it against a wordlist of commonly used and publicly known secret keys (sourced from books, GitHub, StackOverflow and various other sources).

[Flask-unsign](https://pypi.org/project/flask-unsign/) can do that for us

