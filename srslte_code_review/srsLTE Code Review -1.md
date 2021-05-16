# srsLTE Code Review -1

We have a research on the sib data block. The whole name of SIB is the SystemInfomationBlock.

We started by the sib_type1_s.

Here we provide the origin code:

```c++
uint32_t rrc::generate_sibs()
{
  // nof_messages includes SIB2 by default, plus all configured SIBs
  uint32_t           nof_messages = 1 + cfg.sib1.sched_info_list.size();
  sched_info_list_l& sched_info   = cfg.sib1.sched_info_list;

  // Store configs,SIBs in common cell ctxt list
  cell_common_list.reset(new cell_info_common_list{cfg});

  // generate and pack into SIB buffers
  for (uint32_t cc_idx = 0; cc_idx < cfg.cell_list.size(); cc_idx++) {
    cell_info_common* cell_ctxt = cell_common_list->get_cc_idx(cc_idx);
    // msg is array of SI messages, each SI message msg[i] may contain multiple SIBs
    // all SIBs in a SI message msg[i] share the same periodicity
    asn1::dyn_array<bcch_dl_sch_msg_s> msg(nof_messages + 1);

    // Copy SIB1 to first SI message
    msg[0].msg.set_c1().set_sib_type1() = cell_ctxt->sib1;

    // Copy rest of SIBs
    for (uint32_t sched_info_elem = 0; sched_info_elem < nof_messages - 1; sched_info_elem++) {
      uint32_t msg_index = sched_info_elem + 1; // first msg is SIB1, therefore start with second

      msg[msg_index].msg.set_c1().set_sys_info().crit_exts.set_sys_info_r8();
      sys_info_r8_ies_s::sib_type_and_info_l_& sib_list =
          msg[msg_index].msg.c1().sys_info().crit_exts.sys_info_r8().sib_type_and_info;

      // SIB2 always in second SI message
      if (msg_index == 1) {
        sib_info_item_c sibitem;
        sibitem.set_sib2() = cell_ctxt->sib2;
        sib_list.push_back(sibitem);
      }

      // Add other SIBs to this message, if any
      for (auto& mapping_enum : sched_info[sched_info_elem].sib_map_info) {
        sib_list.push_back(cfg.sibs[(int)mapping_enum + 2]);
      }
    }

    // Pack payload for all messages
    for (uint32_t msg_index = 0; msg_index < nof_messages; msg_index++) {
      srslte::unique_byte_buffer_t sib_buffer = srslte::allocate_unique_buffer(*pool);
      asn1::bit_ref                bref(sib_buffer->msg, sib_buffer->get_tailroom());
      if (msg[msg_index].pack(bref) == asn1::SRSASN_ERROR_ENCODE_FAIL) {
        rrc_log->error("Failed to pack SIB message %d\n", msg_index);
      }
      sib_buffer->N_bytes = bref.distance_bytes();
      cell_ctxt->sib_buffer.push_back(std::move(sib_buffer));

      // Log SIBs in JSON format
      std::string log_msg("CC" + std::to_string(cc_idx) + " SIB payload");
      log_rrc_message(
          log_msg, Tx, cell_ctxt->sib_buffer.back().get(), msg[msg_index], msg[msg_index].msg.c1().type().to_string());
    }

    if (cfg.sibs[6].type() == asn1::rrc::sys_info_r8_ies_s::sib_type_and_info_item_c_::types::sib7) {
      sib7 = cfg.sibs[6].sib7();
    }
  }

  return nof_messages;
}
```

In our research we need to know how the program encode sib1 and how it store in the program.

**Store:**

```C++
/*rrc.h*/
class rrc
{
  //...code...
  std::unique_ptr<cell_info_common_list> cell_common_list;
  //...code...
}
```

"cell_info_common_list" is a kind of user defined class.

```c++
/*rrc_cell_cfg.h*/
class cell_info_common_list
{
public:
  explicit cell_info_common_list(const rrc_cfg_t& cfg_);

  cell_info_common*       get_cc_idx(uint32_t enb_cc_idx) { return cell_list[enb_cc_idx].get(); }
  const cell_info_common* get_cc_idx(uint32_t enb_cc_idx) const { return cell_list[enb_cc_idx].get(); }
  const cell_info_common* get_cell_id(uint32_t cell_id) const;
  const cell_info_common* get_pci(uint32_t pci) const;
  size_t                  nof_cells() const { return cell_list.size(); }

private:
  const rrc_cfg_t&                                cfg;
  std::vector<std::unique_ptr<cell_info_common> > cell_list;//Important data member
};
```

Exactly, "cell_info_common" is also a kind of struct:

```C++
/*rrc_cell_cfg.h*/
struct cell_info_common {
  uint32_t                                  enb_cc_idx = 0;
  asn1::rrc::mib_s                          mib;
  asn1::rrc::sib_type1_s                    sib1;
  asn1::rrc::sib_type2_s                    sib2;
  const cell_cfg_t&                         cell_cfg;
  std::vector<srslte::unique_byte_buffer_t> sib_buffer; ///< Packed SIBs for given CC
  std::vector<const cell_info_common*>      scells;

  cell_info_common(uint32_t idx_, const cell_cfg_t& cfg) : enb_cc_idx(idx_), cell_cfg(cfg) {}
};
```

We find the sib1 ,it is what we want.