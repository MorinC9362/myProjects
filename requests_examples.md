# Requests Examples

A couple of examples of funtions that use the requests module to make api requests.

## Example 1

```python

import logging
import requests

from functools import wraps

from .config import ConfigAPI
from .logger import Logger


class API(ConfigAPI):

    def __init__(self, sport=None, league=None):
        super().__init__(sport, league)
        if self.DEV:
            logger = Logger()
            logger.run_logger()
            logging.info("API Configuration Loaded")

    def espn_api_call(self, endpoint=None, query=None, player_id=None):
        url = self.espn_player_url if player_id else self.espn_base_url
        url = endpoint and "/".join([url, endpoint]) or url
        url = player_id and "/".join([url, str(player_id)]) or url
        req = requests.get(url, query)
        logging.info(f"requested url: {req.url}")
        return req

    def cfbd_api_call(self, endpoint=None, query=None):
        url = endpoint and "/".join([self.cfbd_base_url, endpoint]) or self.cfbd_base_url
        req = requests.get(url, query, headers=self.cfbd_headers)
        return req

    def server_api_call(self, endpoint=None, query=None, params=None):
        url = endpoint and "/".join([self.server_base_url, endpoint]) or self.server_base_url
        url = query and "/".join([url, query]) or url
        req = requests.get(url, headers=self.server_headers, params=params)
        logging.info(f"requested url: {req.url}")
        return req
```

## Example 2

This one may be a bit more confusing—its extracted from a larger program: [AutoMod Chat CLient](https://github.com/deoncarlette/AutoMod/blob/main/automod/chat.py)

```python
class MW(ChatConfig):

    def __init__(self, account, config):
        self.mw_message_responded_set = set()
        self.mw_defined_term_set = set()
        super().__init__(account, config)

    def __str__(self):
        pass

    def run_mw_dict_client(self, mw_requests, channel, delay=30):

        filtered_requests = self.filter_new_requests(mw_requests)
        if not filtered_requests:
            logging.info("Responses have already been sent for all mw requests")
            return

        for request in filtered_requests:
            logging.info(f"mw_requests: {request}")
            message = request.get("message")
            term = self.extract_term(message)
            logging.info(f"mw_requests: {term}")
            undefined_term = term if term not in self.mw_defined_term_set else None

            if not undefined_term:
                logging.info(f"[{term}] has already been defined")
                continue

            defined_term = self.get_definition(term)
            definition = self.clean_definition(defined_term)

            message_id = request.get("message_id")
            user_name = request.get("user_profile").get("name")

            response = self.set_response(user_name, term, definition)
            send = self.send_command_response(channel, response, delay)

            if send:
                self.mw_defined_term_set.add(term)
                self.mw_message_responded_set.add(message_id)

    def filter_new_requests(self, mw_requests):
        filtered_requests = [_ for _ in mw_requests if _.get("message_id") not in self.mw_message_responded_set]
        return filtered_requests

    @staticmethod
    def extract_term(message):
        term = message.split("/definition: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/definition:")
        if len(term) > 1:
            return term[1]
        term = message.split("/definition ")
        if len(term) > 1:
            return term[1]
        term = message.split("/define: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/define:")
        if len(term) > 1:
            return term[1]
        term = message.split("/define ")
        if len(term) > 1:
            return term[1]
        term = message.split("/def: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/def:")
        if len(term) > 1:
            return term[1]
        term = message.split("/def ")
        if len(term) > 1:
            return term[1]
        term = message.split("/dictionary: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/dictionary:")
        if len(term) > 1:
            return term[1]
        term = message.split("/dictionary ")
        if len(term) > 1:
            return term[1]
        term = message.split("/dict: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/dict:")
        if len(term) > 1:
            return term[1]
        term = message.split("/dict ")
        if len(term) > 1:
            return term[1]
        term = message.split("/mw: ")
        if len(term) > 1:
            return term[1]
        term = message.split("/mw:")
        if len(term) > 1:
            return term[1]
        term = message.split("/mw ")
        if len(term) > 1:
            return term[1]

    def get_definition(self, term):

        @validate_response
        def api_request():

            req = requests.get(f"{self.MW_URL}{term}?key={self.MW_KEY}")
            return req

        return api_request()

    @staticmethod
    def clean_definition(definition):

        defined = "Sorry, something happened. Please try again."

        if isinstance(definition[0], dict):
            defined = [_.get("shortdef") for _ in definition][0]
            logging.info(defined)
            if defined:
                defined = defined[0]

        elif isinstance(definition, list):
            definition = definition
            word_list = ", ".join(definition[:5])
            defined = f"Sorry, I couldn't define your word. \
                      You may want to try again with one of the following: {word_list}, or {definition[6]}."
            logging.info(defined)

        logging.info(f"{defined}")

        return defined

    @staticmethod
    def set_response(user_name, term, definition):

        term = f"[Merriam-Webster] {term}"
        term = fancy.bold_serif(term)

        reply_message = f"@{user_name} {term}—{definition}"

        if len(reply_message) > 250:
            reply_message = reply_message[:250].rsplit(".", 1)[0] + "."

        logging.info(reply_message)
        return reply_message
        
```
