use anchor_lang::prelude::*;

#[program]
mod governance {
    use super::*;

    pub fn create_proposal(ctx: Context<CreateProposal>, description: String) -> Result<()> {
        let proposal = &mut ctx.accounts.proposal;
        proposal.description = description;
        proposal.votes_for = 0;
        proposal.votes_against = 0;
        proposal.executed = false;
        Ok(())
    }

    pub fn vote(ctx: Context<Vote>, support: bool) -> Result<()> {
        let proposal = &mut ctx.accounts.proposal;
        let voter = &ctx.accounts.voter;

        require!(!proposal.executed, CustomError::ProposalExecuted);
        require!(!proposal.voters.contains(&voter.key()), CustomError::AlreadyVoted);
        
        proposal.voters.push(voter.key());
        if support {
            proposal.votes_for += 1;
        } else {
            proposal.votes_against += 1;
        }
        Ok(())
    }

    pub fn execute_proposal(ctx: Context<ExecuteProposal>) -> Result<()> {
        let proposal = &mut ctx.accounts.proposal;
        require!(proposal.votes_for > proposal.votes_against, CustomError::NotEnoughSupport);
        require!(!proposal.executed, CustomError::ProposalExecuted);
        
        proposal.executed = true;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct CreateProposal<'info> {
    #[account(init, payer = user, space = 256)]
    pub proposal: Account<'info, Proposal>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Vote<'info> {
    #[account(mut)]
    pub proposal: Account<'info, Proposal>,
    #[account(mut)]
    pub voter: Signer<'info>,
}

#[derive(Accounts)]
pub struct ExecuteProposal<'info> {
    #[account(mut)]
    pub proposal: Account<'info, Proposal>,
}

#[account]
pub struct Proposal {
    pub description: String,
    pub votes_for: u64,
    pub votes_against: u64,
    pub executed: bool,
    pub voters: Vec<Pubkey>,
}

#[error_code]
pub enum CustomError {
    #[msg("Proposal already executed.")]
    ProposalExecuted,
    #[msg("Not enough support to execute proposal.")]
    NotEnoughSupport,
    #[msg("You have already voted on this proposal.")]
    AlreadyVoted,
}
кфку
