# Aleo-Test
program workshopnamespacelongisrequired.aleo {
  mapping account: address => u64;

  record token {
    owner: address,
    amount: u64,
  }

  async transition mint_public (public receiver: address, public amount: u64) -> Future {


    return finalize_mint_public(receiver, amount);
  }

  async function finalize_mint_public (public receiver: address, public amount: u64) {
    let current_amount: u64 = Mapping::get_or_use(account, receiver, 0u64);

    Mapping::set(account, receiver, current_amount + amount);
  }

  transition mint_private (receiver: address, amount: u64) -> token {
    return token {
      owner: receiver,
      amount: amount,
    };
  }

  async transition transfer_public (public receiver: address, public amount: u64) -> Future {
    return update_public_transfer(self.caller, receiver, amount);
  }

  async function update_public_transfer (public sender: address, public receiver: address, public amount: u64) {
    let sender_amount: u64 = Mapping::get_or_use(account, sender, 0u64);

    Mapping::set(account, sender, sender_amount - amount);

    let public_receiver: u64 = Mapping::get_or_use(account, receiver, 0u64);

    Mapping::set(account, receiver, public_receiver + amount);
  }

  transition transfer_private (sender_token: token, receiver: address, amount: u64) -> (token, token) {
    let remaining: token = token {
      owner: sender_token.owner,
      amount: sender_token.amount - amount,
    };

    let transferred: token = token {
      owner: receiver,
      amount: amount,
    };

    return (remaining, transferred);
  }

  async transition transfer_private_to_public (sender_token: token, public receiver: address, public amount: u64) -> (token, Future) {
    let remaining: token = token {
      owner: sender_token.owner,
      amount: sender_token.amount - amount,
    };

    return (remaining, finalize_private_to_public(receiver, amount));
  }

  async function finalize_private_to_public (public receiver: address, public amount: u64) {
    let public_receiver: u64 = Mapping::get_or_use(account, receiver, 0u64);

    Mapping::set(account, receiver, public_receiver + amount);
  }

  async transition transfer_public_to_private (receiver: address, amount: u64) -> (token, Future) {
    let transferred: token = token {
      owner: receiver,
      amount: amount,
    };

    return (transferred, finalize_transfer_public_to_private(self.caller, amount));
  }

  async function finalize_transfer_public_to_private (public sender: address, public amount: u64) {
    let current_amount: u64 = Mapping::get_or_use(account, sender, 0u64);

    Mapping::set(account, sender, current_amount - amount);
  }
}
